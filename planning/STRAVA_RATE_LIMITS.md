# Strava API Rate Limits — Risk Assessment

## Your Current Limits

| Limit | Window | Value |
|---|---|---|
| Overall requests | 15 minutes | 600 |
| Overall requests | Daily | 6,000 |
| **Read requests** | **15 minutes** | **300** |
| **Read requests** | **Daily** | **3,000** |
| Connected athletes | — | 999 |

The read limits are the binding constraint — every activity processed by the webhook Lambda makes exactly **1 read API call** to `GET /api/v3/activities/{id}`.

---

## How Many Strava API Calls Does the Current Integration Make?

### Per activity event (webhook `aspect_type: "create"`)

| Step | Strava API call? | Endpoint | Rate limit category |
|---|---|---|---|
| Dedup check | No | RDS only | — |
| Get tokens from DB | No | RDS only | — |
| Token refresh (if expired) | **Sometimes** | `POST /oauth/token` | **Not counted** in read limits |
| Fetch activity details | **Always** | `GET /api/v3/activities/{id}` | ✅ Counts against Read limit |
| Insert into DB / update periods | No | RDS only | — |

**The bottom line: every new activity = 1 read API call.**

Token refreshes call `/oauth/token`, which is an OAuth endpoint — Strava does not count this against the read rate limit. Strava access tokens expire every 6 hours, so athletes who upload once a day will trigger at most one refresh per day each.

---

## When Will You Hit the Limits?

### Daily limit — 3,000 read calls/day

| Connected athletes | Avg activities/athlete/day | Daily calls | Safe? |
|---|---|---|---|
| 100 | 1 | 100 | ✅ |
| 500 | 1 | 500 | ✅ |
| 999 | 1 | 999 | ✅ |
| 999 | 2 | 1,998 | ✅ |
| 999 | 3 | 2,997 | ⚠️ Right at the cap |
| 999 | 4 | 3,996 | ❌ Over by 33% |

With 999 athletes, you hit the daily limit when the average athlete logs more than **3 activities per day**. This is realistic for triathletes, people who commute by bike and also run in the evening, or very active members — not a theoretical edge case.

### 15-minute limit — 300 read calls per 15 minutes

This is the **more dangerous** limit and the one most likely to cause problems first.

The 15-minute window doesn't care about your daily average — it cares about **bursts**. Strava users don't upload activities evenly throughout the day. They upload after they finish, and large groups of athletes often finish at the same time.

| Scenario | Athletes uploading | Burst window | Calls in peak 15 min | Safe? |
|---|---|---|---|---|
| Normal weekday, spread across morning | 100 | 2 hours | ~12 | ✅ |
| Saturday morning runs, spread | 300 | 2 hours | ~37 | ✅ |
| Parkrun — 400 athletes, finish in 30 min | 400 | 30 min | **~200** | ⚠️ Getting close |
| Local charity 5K — 500 athletes auto-upload | 500 | 15–20 min | **300–500** | ❌ Likely over |
| Mass event — 800 athletes, auto-upload enabled | 800 | 15 min | **800** | ❌ Hit and exceeded |

**The 15-minute window is the real risk.** You don't need 999 athletes to hit it — you need ~300 athletes all finishing and uploading within the same 15-minute window. A popular local race or parkrun participation spike would trigger this.

---

## The Bigger Problem: Silent Activity Drops

Regardless of what the rate limits are, there is a **critical bug in the current implementation** that means hitting a rate limit causes permanent, silent data loss.

### What happens when the 300/15min limit is hit

When Strava returns a `429 Too Many Requests` to the Lambda:

```typescript
// strava-webhook/index.ts — processActivity()
const activityResponse = await fetch(
  `https://www.strava.com/api/v3/activities/${stravaActivityId}`,
  { headers: { Authorization: `Bearer ${tokens.access_token}` } }
);

if (!activityResponse.ok) {
  console.error(`Failed to fetch activity ${stravaActivityId}:`, await activityResponse.text());
  return; // ← exits silently
}
```

The function returns early. No dedup record is ever written to `strava_activities` (that only happens after a successful fetch). Then in the handler:

```typescript
try {
  await processActivity(body);
} catch (err) {
  console.error("Error processing activity:", err);
  // Still return 200 so Strava doesn't retry excessively ← ⚠️
}

return { statusCode: 200, body: "OK" }; // ← always 200
```

### The consequence

1. Lambda receives webhook, hits 429 from Strava API
2. `processActivity` returns without inserting a dedup record
3. Lambda returns **200 OK** to Strava
4. Strava sees 200 → considers the event delivered → **never retries**
5. No dedup record exists, so there's no way to know the activity was missed
6. **The activity is permanently lost** from the user's goal progress

This happens silently with no alert, no user notification, and no recovery path.

The comment "Still return 200 so Strava doesn't retry excessively" is well-intentioned for the error case, but the problem is it also swallows 429s — which are retryable and should trigger a retry.

---

## Summary

| Risk | Likelihood at current scale | Likelihood at 999 athletes |
|---|---|---|
| Hit 15-min read limit | Low (unless event-day spike) | Moderate — any group event |
| Hit daily read limit | Very low | Moderate — active users |
| Silent activity loss if limit hit | **Guaranteed** when limit hit | **Guaranteed** when limit hit |

The rate limits themselves are manageable for now, but the combination of **no queue, no retry, and always-200** means that when limits are eventually hit (and they will be), users lose goal progress with no warning and no way to recover it.

---

## Recommended Fixes (Priority Order)

### 1. ✅ Don't swallow 429s — return a non-200 to Strava (done)

`processActivity` now throws a `RATE_LIMITED:` error when Strava returns 429. The handler catches it, logs it, and returns `{ statusCode: 500 }` — which tells Strava the event was not delivered and triggers its retry mechanism (exponential backoff, up to 2 days).

All other errors still return 200 to Strava to avoid indefinite retries on unrecoverable failures (bad token, unknown athlete, etc.).

The dedup check at the top of `processActivity` (`strava_activities` table) ensures that if Strava redelivers a webhook after the rate limit window resets, the activity is not double-counted.

### 2. ✅ Track rate limit headers — CloudWatch EMF metrics (done)

After every successful `GET /api/v3/activities/{id}` call, the Lambda now reads Strava's rate limit response headers and logs them using **CloudWatch Embedded Metric Format (EMF)**. CloudWatch automatically converts the EMF log lines into queryable metrics — no extra SDK, no `PutMetricData` calls.

**Metrics emitted to `GoalsClub/Strava` namespace:**
- `RateLimitUsed15Min` — overall requests used in the current 15-min window
- `ReadRateLimitUsed15Min` — read requests used in the current 15-min window
- `RateLimitUsedDaily` — overall requests used today
- `ReadRateLimitUsedDaily` — read requests used today

**Warnings:** a `console.warn` is emitted when any 15-minute window or the daily read limit exceeds **80%** of capacity. These appear in CloudWatch Logs and can be used to trigger a CloudWatch alarm.

**Cost:** EMF metrics are ~$0.30 per metric per month after the first 10 free custom metrics.

### 3. ✅ SQS decoupling — receiver + processor split (done)

The webhook Lambda has been split into two:

**Receiver** (`strava-webhook/index.ts`) — thin, fast. Validates the Strava subscription challenge (GET) or enqueues the event to SQS (POST) and immediately returns 200. Never calls the Strava API.

**Processor** (`strava-webhook-processor/index.ts`) — triggered by SQS. Contains all the activity processing logic (dedup, token refresh, Strava API call, DB writes). On a 429 rate limit response it throws, which causes SQS to retry after the visibility timeout.

**Infrastructure** (`modules/lambda-strava-webhook-processor.ts`):
- `StravaActivityQueue` — Standard SQS queue, 120s visibility timeout, 2-day retention, max 5 retries before DLQ
- `StravaActivityDLQ` — Dead-letter queue, 14-day retention for manual inspection
- `StravaWebhookProcessorLambda` — 90s timeout, unreserved concurrency (see note below)
- SQS event source mapping with `ReportBatchItemFailures` so individual message failures don't block the whole batch

**Rate limiting benefit:** SQS queues activities so burst events (race day) don't all hit the Strava API simultaneously. The queue acts as a natural buffer — messages are processed as fast as the Lambda can handle them rather than all at once.

**Concurrency note — `reservedConcurrentExecutions` deliberately not set:**
Setting `reservedConcurrentExecutions: 5` on the processor Lambda was the original design to throttle concurrent Strava API calls and stay within the 300 read calls/15min limit. However, AWS accounts have a minimum of 10 **unreserved** concurrent executions that must always remain available. On smaller accounts (low total concurrency limit, e.g. 50), reserving 5 for this Lambda can breach that floor and causes a Pulumi deploy error:

```
InvalidParameterValueException: Specified ReservedConcurrentExecutions for function
decreases account's UnreservedConcurrentExecution below its minimum value of [10].
```

**When to add it back:**
1. Run `aws lambda get-account-settings --region eu-west-1` to check `UnreservedConcurrentExecutions`
2. If `UnreservedConcurrentExecutions - 5 >= 10`, it's safe to add `reservedConcurrentExecutions: 5`
3. Add it when CloudWatch metrics (`ReadRateLimitUsed15Min`) show approaching 80% of the 300/15min limit

**Cost:** SQS Standard queue + extra Lambda invocations = effectively £0 at current athlete count (under 1M SQS requests/month free tier).

### 4. ✅ Proactive token refresh Lambda (done)

A scheduled Lambda (`StravaTokenRefreshLambda`) runs every **5 hours** via an EventBridge rule. It queries `strava_tokens` for any token with `expires_at <= now + 6 hours` and refreshes them via `POST /oauth/token`.

**Files:**
- `packages/infra/lambdas/strava-token-refresh/index.ts` — Lambda handler
- `packages/infra/modules/lambda-strava-token-refresh.ts` — Pulumi IAM, Lambda, EventBridge schedule

**Why 5 hours / 6-hour look-ahead:** Strava tokens last exactly 6 hours. Running every 5 hours with a 6-hour refresh window ensures tokens are always refreshed with a 1-hour margin, even if a scheduled run is delayed.

**Cost:** EventBridge rule + Lambda = effectively £0 at any scale (132 scheduled invocations/month, within free tier).

