# Garmin Connect Integration — Planning Doc

Last updated: 2026-05-17

> **Status:** Planning — not yet started. Requires Garmin Health API application approval.

---

## Overview

Add Garmin Connect as the second fitness platform alongside Strava. Users with Garmin wearables (Forerunner, Fenix, Venu, vívoactive, etc.) can connect their account to auto-sync activities to Goals Club goals.

**Why Garmin:**
- Second-largest fitness platform after Strava for endurance athletes
- Many runners/walkers use Garmin watches without Strava
- Push-based webhook system (similar to Strava) — no polling needed
- Free API access (no per-call fees)
- Broader data than Strava: steps, sleep, stress, body composition (useful for future habit goals)

---

## Garmin Health API — Access Requirements

### Application Process

| Requirement | Status | Notes |
|-------------|--------|-------|
| **Business entity** | ⚠️ Required | Garmin requires a registered business. Sole trader or LTD company accepted |
| **Live website** | ✅ Ready | thegoalsclub.co.uk |
| **Privacy policy** | ✅ Ready | `/privacy` — will need Garmin-specific data handling section |
| **Terms of service** | ✅ Ready | `/terms` |
| **Application form** | 🔜 Pending | Apply at [developer.garmin.com/health-api](https://developer.garmin.com/health-api/) |
| **Use case description** | 🔜 Pending | "Fitness goal tracking — users connect Garmin to auto-log activities against recurring and collection goals" |
| **Review timeline** | — | Typically 2-4 weeks |
| **Cost** | Free | No per-call fees or subscription |

### What to Include in Application

```
App Name: The Goals Club
Website: https://thegoalsclub.co.uk
Description: Social fitness goal tracking platform. Users set personal goals
(e.g. "Run 100 miles this month", "Complete all Wainwrights") and track progress
via manual logging or fitness platform auto-sync. Garmin integration would
automatically log activities from Garmin devices to matching user goals based on
activity type and linked goal mappings.

Data usage:
- Activity type, distance, duration, date — matched to user goals for progress tracking
- GPS start coordinates (optional) — for location-based goal item claiming
- Route polyline (optional) — displayed on activity detail cards
- Steps/daily summaries (future) — for step-count and walking distance goals

Data stored in encrypted Aurora MySQL database. Users can disconnect at any time
via Settings page, which deletes all stored tokens and Garmin data.
```

---

## Technical Architecture

### Comparison with Strava Implementation

| Component | Strava (Current) | Garmin (Proposed) |
|-----------|-----------------|-------------------|
| **Auth protocol** | OAuth 2.0 | OAuth 1.0a (3-legged) |
| **Token storage** | `strava_tokens` table | `garmin_tokens` table |
| **Webhook delivery** | POST to API Gateway | POST to API Gateway (Garmin calls it "Ping") |
| **Webhook format** | `{ object_type, aspect_type, object_id }` | `{ activities: [{ userId, summaryId }] }` (batch) |
| **Activity fetch** | `GET /api/v3/activities/{id}` | Activity data included in webhook payload (no separate fetch needed!) |
| **Token refresh** | Refresh token exchange | OAuth 1.0a tokens don't expire (no refresh needed!) |
| **Rate limits** | 300 reads/15min, 3000/day | More generous (per-endpoint, typically ~1000/15min) |
| **Dedup key** | `strava_activity_id` (BIGINT) | `garmin_activity_id` (STRING) |
| **Activity types** | Run, Walk, Ride, Swim, etc. | Running, Walking, Cycling, Swimming, etc. (different naming) |

### Key Simplifications vs Strava

1. **No token refresh Lambda needed** — OAuth 1.0a access tokens don't expire
2. **No activity fetch Lambda needed** — Garmin pushes full activity data in the webhook (unlike Strava which only sends an ID)
3. **Simpler rate limit handling** — more generous limits + less API calls needed
4. **Batch webhook processing** — Garmin sends multiple activities per webhook call

### Key Complications vs Strava

1. **OAuth 1.0a** — more complex handshake (request token → authorize → access token), requires HMAC-SHA1 signature on every request
2. **Different activity type naming** — need a mapping layer (`Running` → `Run`, `Walking` → `Walk`, etc.)
3. **Garmin user ID** — string-based (vs Strava's numeric athlete ID)

---

## Data Model

### New Table: `garmin_tokens`

| Column | Type | Notes |
|--------|------|-------|
| id | VARCHAR(26) | PK (ULID) |
| user_id | VARCHAR(26) | FK → users, UNIQUE, NOT NULL, CASCADE |
| garmin_user_id | VARCHAR(128) | NOT NULL |
| access_token | VARCHAR(255) | NOT NULL |
| access_token_secret | VARCHAR(255) | NOT NULL (OAuth 1.0a requires token + secret) |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Indexes:** user_id (unique), garmin_user_id

### New Table: `garmin_activities`

| Column | Type | Notes |
|--------|------|-------|
| id | VARCHAR(26) | PK (ULID) |
| garmin_activity_id | VARCHAR(128) | UNIQUE, NOT NULL |
| user_id | VARCHAR(26) | FK → users, NOT NULL, CASCADE |
| garmin_activity_type | VARCHAR(50) | |
| distance_meters | FLOAT | |
| moving_time_seconds | INT | |
| start_date | DATETIME | |
| name | VARCHAR(255) | |
| start_lat | DECIMAL(10,8) | GPS latitude |
| start_lng | DECIMAL(11,8) | GPS longitude |
| processed_at | DATETIME | |
| created_at | DATETIME | |

### Extend Existing Tables

**`user_goal_activities`** — already has `strava_activity_id` FK. Add:

| Column | Type | Notes |
|--------|------|-------|
| garmin_activity_id | VARCHAR(26) | FK → garmin_activities, SET NULL |

**`strava_goal_links`** → Rename/generalise to `fitness_goal_links` or create parallel `garmin_goal_links`:

| Column | Type | Notes |
|--------|------|-------|
| id | VARCHAR(26) | PK |
| user_id | VARCHAR(26) | FK → users |
| user_goal_id | VARCHAR(26) | FK → user_goals |
| garmin_activity_type | VARCHAR(50) | e.g. "Running", "Walking" |
| created_at | DATETIME | |

---

## Infrastructure

### New AWS Resources

| Resource | Purpose | Module |
|----------|---------|--------|
| **Lambda: garmin-oauth** | OAuth 1.0a handshake (request token → access token exchange) | `lambda-garmin-oauth.ts` |
| **Lambda: garmin-webhook** | Receiver — validates + enqueues to SQS | `lambda-garmin-webhook.ts` |
| **Lambda: garmin-webhook-processor** | SQS consumer — processes activity data, matches goals, logs progress | `lambda-garmin-webhook-processor.ts` |
| **SQS Queue + DLQ** | Decouple webhook from processing (same pattern as Strava) | Part of processor module |
| **API Gateway route** | `POST /garmin` webhook endpoint | Added to existing API Gateway |
| **CloudWatch Alarms** | DLQ depth + Lambda errors (extend existing alarm module) | `cloudwatch-alarms.ts` |

**NOT needed (vs Strava):**
- ❌ Token refresh Lambda — OAuth 1.0a tokens don't expire
- ❌ EventBridge scheduled rule — no refresh cycle needed

### Lambda Architecture

```
Garmin Health API
    │
    ▼ (Ping/webhook — includes activity data)
API Gateway (POST /garmin)
    │
    ▼
Garmin Webhook Receiver Lambda
    │ (validate, enqueue)
    ▼
SQS Queue ──▶ DLQ (after 5 retries)
    │
    ▼
Garmin Webhook Processor Lambda
    │ (dedup, match goal links, log activity, update periods/streaks)
    ▼
Aurora MySQL (garmin_activities, user_goal_activities, user_goal_periods)
```

---

## GraphQL Schema Additions

```graphql
# Mutations
connectGarmin(requestToken: String!, verifier: String!): Boolean! @aws_cognito_user_pools
disconnectGarmin: Boolean! @aws_cognito_user_pools
getGarminRequestToken: GarminRequestToken! @aws_cognito_user_pools

# Types
type GarminRequestToken {
  requestToken: String!
  authorizeUrl: String!
}

# Extend existing
type UserSettings {
  # ...existing fields...
  garminConnected: Boolean!
  garminUserId: String
}
```

---

## Web App Changes

### Settings Page
- Add "Connect Garmin" button alongside existing "Connect Strava"
- Garmin-branded button (Garmin blue #007DC5)
- Same pattern: OAuth flow → redirect → callback → show connected state
- "Disconnect Garmin" with confirmation modal

### Goal Link Setup
- When user has Garmin connected, show Garmin activity types in goal link setup (alongside Strava types)
- Activity type mapping: `Running` ↔ `Run`, `Walking` ↔ `Walk`, `Cycling` ↔ `Ride`, `Swimming` ↔ `Swim`, etc.

### Activity Cards
- Show Garmin attribution badge (like "Powered by Strava" but for Garmin)
- Same route map display for GPS activities

---

## Activity Type Mapping

| Garmin Type | Strava Equivalent | Goals Club Internal |
|-------------|-------------------|---------------------|
| `running` | `Run` | `Run` |
| `walking` | `Walk` | `Walk` |
| `cycling` | `Ride` | `Ride` |
| `swimming` | `Swim` | `Swim` |
| `hiking` | `Hike` | `Hike` |
| `trail_running` | `Trail Run` | `Trail Run` |
| `open_water_swimming` | `Open Water Swim` | `Open Water Swim` |
| `strength_training` | `Weight Training` | `Weight Training` |
| `yoga` | `Yoga` | `Yoga` |

Consider: normalise to an internal activity type enum so goal links work across both platforms.

---

## Implementation Order

| Step | Effort | Dependency |
|------|--------|------------|
| 1. Apply for Garmin Health API access | 0 days | Business registration |
| 2. Database migration (2 new tables + FK) | 0.5 days | Approval received |
| 3. GraphQL schema + resolvers (connect/disconnect) | 1 day | — |
| 4. OAuth 1.0a Lambda (request token + access token exchange) | 1-2 days | Garmin API credentials |
| 5. Webhook receiver Lambda | 0.5 days | — |
| 6. SQS + DLQ + webhook processor Lambda | 1-2 days | — |
| 7. Infra modules (Pulumi) | 1 day | — |
| 8. Web: Settings page (connect/disconnect) | 0.5 days | — |
| 9. Web: Goal link setup for Garmin types | 0.5 days | — |
| 10. Activity type mapping layer | 0.5 days | — |
| 11. Garmin webhook subscription registration | 0.5 days | Webhook endpoint deployed |
| 12. Testing + deploy | 1 day | — |
| **Total** | **~7-9 days** | — |

---

## Compliance & Branding

| Requirement | Action |
|-------------|--------|
| **Garmin branding** | Use official Garmin Connect logo/badge, follow brand guidelines |
| **Attribution** | "Connected with Garmin" badge on Settings page and activity cards |
| **Privacy policy** | Add Garmin-specific section covering data access, storage, deletion |
| **Disconnect flow** | One-click disconnect in Settings — deletes tokens + optionally clears synced activities |
| **Data retention** | Clearly state what Garmin data is stored and for how long |

---

## Future: Abstract Fitness Provider Layer

When adding a third platform (Fitbit, Apple Health), consider abstracting the fitness provider integration:

```
providers/
├── strava/
│   ├── oauth.ts
│   ├── webhook.ts
│   └── activity-mapper.ts
├── garmin/
│   ├── oauth.ts
│   ├── webhook.ts
│   └── activity-mapper.ts
└── shared/
    ├── activity-types.ts     # Canonical activity type enum
    ├── goal-link-matcher.ts  # Match activity → goal
    └── progress-logger.ts    # Log activity to user_goal_activities
```

This would make adding new providers much faster (only implement OAuth + webhook adapter, reuse all goal matching/progress logic).

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Garmin rejects API application | Ensure business registration is complete before applying. Reference live Strava integration as proof of concept |
| OAuth 1.0a complexity | Use established library (e.g. `oauth-1.0a` npm package). Well-documented protocol |
| Activity type mismatch | Build mapping layer early. Start with core types (Run, Walk, Ride, Swim) and add others based on user data |
| Users with both Strava + Garmin | Dedup by timestamp + distance — if Garmin and Strava report the same activity (common for users who sync Garmin → Strava), don't double-count |
| Garmin API changes | Subscribe to Garmin developer newsletter. Their API is mature and stable |

---

## Decision Log

| Decision | Answer | Rationale |
|----------|--------|-----------|
| Separate tables (not shared with Strava) | Yes | Different auth model (OAuth 1.0a vs 2.0), different IDs, cleaner separation |
| Normalised activity types | TBD | Consider when implementing — may want canonical `activity_type` enum in `user_goal_activities` |
| Abstract provider layer | Defer | Premature until we have 2+ providers. Build Garmin as a parallel to Strava, refactor later if adding a third |
| Webhook-only (no polling/manual sync) | Start with webhook | Garmin pushes data proactively. Add manual "Sync History" later if users request it |

