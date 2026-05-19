# Next Steps

Last updated: 2026-05-19

---

## 📍 Current Phase: Soft Launch (Friends Beta) — Dual Environment

The app is live in both environments:
- **Prod**: `thegoalsclub.co.uk` (public-facing, Strava webhooks active)
- **Dev**: `dev.thegoalsclub.co.uk` (development/testing)
- **Admin**: `admin.thegoalsclub.co.uk` (prod) / `dev-admin.thegoalsclub.co.uk` (dev)

Strava is approved for 999 athletes. See [archive/NEXT_STEPS_PRE_LAUNCH.md](./archive/NEXT_STEPS_PRE_LAUNCH.md) for full build history.

---

## 🔜 Up Next

### Strava History Sync — Timeout Fix (URGENT)
The `syncStravaHistory` Lambda times out for users with large activity histories (216 activities failed; incoming user has 5,500+). AppSync has a hard 30s timeout for Lambda resolvers regardless of Lambda timeout.

**Options:**
1. **Async sync** — mutation triggers Lambda asynchronously (returns immediately), frontend polls for completion status via a `syncStatus` query. Requires a `strava_sync_jobs` table or DynamoDB status record.
2. **Incremental sync** — only fetch activities since the last sync (`after` epoch param on Strava API). First sync fetches all; subsequent syncs only fetch new. Reduces volume dramatically.
3. **Both** — async + incremental for the best UX.

### Cognito / Email Polish
- **Cognito hosted UI theming** — brand the login/signup/confirm pages with Goals Club colours and logo
- **Email templates** — style verification code and password reset emails with Goals Club branding
- **Spam guidance** — add a message on the signup confirmation page telling users to check spam/junk folder for the verification code

### UX Fixes
- **Notification bell on mobile** — bell icon shows but clicking it doesn't load notifications. Likely a responsive/routing issue.
- **Getting Started page** — make it user-aware: if logged in, mark off completed sections (e.g. "Connect Strava" ✅ if `stravaConnection.connected`, "Join a goal" ✅ if user has goals)

### Group Challenges — Remaining Polish
- Mobile responsiveness review for group pages
- Group settings page (edit name/description, manage members, remove goals)

---

## ✅ Recently Completed

### May 18, 2026
- **SES production access** — approved by AWS, emails now send from `noreply@thegoalsclub.co.uk` via SES DEVELOPER mode
- **Email deliverability** — added DMARC record, custom MAIL FROM domain (`mail.thegoalsclub.co.uk`) with SPF + MX records. All three auth checks now pass (DKIM ✅, SPF ✅, DMARC ✅)
- **Strava webhook hardening** — processor now handles both `create` and `update` events (safe via INSERT IGNORE dedup). Catches activities that only send `update` webhooks.
- **Strava sync performance** — removed expensive backfill loop from `syncStravaHistory` (was doing N SQL queries per existing activity). Existing activities now skip instantly.
- **Lambda timeout increase** — Strava OAuth Lambda timeout 120s → 300s
- **Strava matched activities** — identified that Strava's "Add People" feature (group activities) does NOT send webhooks to tagged users. Manual sync required for these.

### May 17, 2026
- **Event goal completion via Strava claim** — claiming a Strava activity on an event goal auto-completes it; removing the activity resets to ACTIVE. Two new pipeline resolvers (`claimStravaActivity`, `deleteGoalActivity`), `useDeleteGoalActivity` hook, remove button on GoalDetail
- **AppSync pipeline patterns documented** — `ctx.prev.result` vs `ctx.stash` clarified in APPSYNC_JS_RUNTIME.md
- **Group Challenges Phase 1** — full groups system deployed to dev + prod (3 tables, 12 unit resolvers, 4 pipeline resolvers, leaderboard with monthly filtering, invite link join flow, Add Goal modal, test data seeder)
- **PR review fixes** across data, web, and admin repos (secret redaction, accessibility, pagination, scroll restoration, Cognito logout, lazy Strava maps, Makefile hardening)

### May 16, 2026
- **CloudWatch Alarms** — DLQ depth, Lambda errors, Strava rate limit 80%
- **AWS Budget Alerts** — $25/$50/$75 thresholds
- **Production stack deployment** — all three repos deploy to dev + prod
- **Strava webhook** — prod-only subscription
- **UX improvements** — Explore page polish, RDS cold-start auto-retry
- **Cost optimization** — removed VPC Endpoint for Secrets Manager (~$14.60/mo saved)

### May 11, 2026
- **Strava activity claiming** — GPS proximity + name similarity scoring
- **Strava historical sync** — paginated fetch of all past activities
- **Strava route maps** — Mapbox GL polyline rendering on claimed items

**Full details:** [archive/COMPLETED_POST_LAUNCH.md](./archive/COMPLETED_POST_LAUNCH.md)

---

## 🔥 Immediate — Feedback-Driven

These will be driven by what testers report. Nothing blocked.

- **Monitor & fix** — watch for bugs, UX friction, and error spikes from real usage
- **Badge images l3–l5** — 13 tracks × 3 levels + goat-l6 prestige. Currently show placeholder. Art asset task — see [BADGE_DESIGN_REFERENCE.md](./BADGE_DESIGN_REFERENCE.md)

---

## 🗺️ Roadmap

### Near-Term (next few weeks)

| Feature | Effort | Notes |
|---------|--------|-------|
| **Group Challenges polish** | 2-3 days | Group settings page, mobile review |
| **Mobile polish** | Ongoing | Address any issues testers report on phone viewports |

### Medium-Term (months 2-3)

| Feature | Effort | Notes |
|---------|--------|-------|
| **Super Goals** | 1 week | Parent/child goal hierarchy. `parent_goal_id` column, auto-progress from children. NT regional goals, Wainwright regions |
| **Group Challenges Phase 2** | 1-2 weeks | Milestones/prizes, badge awards for group achievements. See [GROUP_CHALLENGES.md](./GROUP_CHALLENGES.md) |
| **Pipeline resolver conversions** | Ongoing | Convert `joinGoal`, `followUser` etc. to pipelines when touching them. Tech debt — no user impact |

### Long-Term (months 4+)

| Feature | Notes |
|---------|-------|
| **Garmin integration** | Second fitness platform alongside Strava. See [GARMIN_INTEGRATION.md](./GARMIN_INTEGRATION.md) |
| **Achievement sharing** | Share badges/completions to social media |
| **Premium tier** | Group Challenges Pro (£9.99/mo), analytics dashboard |
| **Public launch** | Marketing site, App Store |
| **Event reviews & ratings** | Community content on events |
| **Training plans** | Linked to events and goals |

---

## 🔧 Tech Debt

| Item | Priority | Notes |
|------|----------|-------|
| Pipeline resolver conversions | Low | 4 candidates remaining (6 identified, 2 done: `claimStravaActivity`, `deleteGoalActivity`). Do when touching those resolvers |
| Admin dashboard stats | Low | Users, goals, activities, badges — nice-to-have |
| Organiser approval workflow | Low | Currently go straight to approved |

---

## 🎨 Design Assets Needed

- **Badge images:** l3, l4, l5 for all 13 animal tracks + goat-l6 (prestige). See [BADGE_DESIGN_REFERENCE.md](./BADGE_DESIGN_REFERENCE.md)
- **Event/organiser logos:** parkrun, Great Run, SuperHalfs, Abbott WMM, T100, London Marathon Events, IRONMAN

---

## 📚 Reference Docs

| Document | Purpose |
|----------|---------|
| [PROD_STACK_DEPLOYMENT.md](./PROD_STACK_DEPLOYMENT.md) | Guide to standing up production stack (✅ completed) |
| [STRAVA_RATE_LIMITS.md](./STRAVA_RATE_LIMITS.md) | Rate limit analysis, all 4 fixes deployed |
| [STRAVA_API_APPROVAL.md](./STRAVA_API_APPROVAL.md) | Strava approval status (✅ approved, 999 athletes) |
| [GROUP_CHALLENGES.md](./GROUP_CHALLENGES.md) | B2B2C feature spec — groups, leaderboards, revenue model |
| [GARMIN_INTEGRATION.md](./GARMIN_INTEGRATION.md) | Garmin Connect integration plan — API access, architecture, implementation |
| [BADGE_DESIGN_REFERENCE.md](./BADGE_DESIGN_REFERENCE.md) | Kawaii animal mascot specs for badge artwork |
| [TECHNICAL_ARCHITECTURE.md](./TECHNICAL_ARCHITECTURE.md) | System architecture overview |
| [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) | Database schema reference |
| [EVENTS_SYSTEM.md](./EVENTS_SYSTEM.md) | Events & organisers system design |
| [SCROLL_RESTORATION.md](./SCROLL_RESTORATION.md) | Scroll restoration implementation notes |
| [APPSYNC_JS_RUNTIME.md](./APPSYNC_JS_RUNTIME.md) | AppSync JS resolver limitations & workarounds |
| [PULUMI_STATE_MANAGEMENT.md](./PULUMI_STATE_MANAGEMENT.md) | Recovery procedures for stale Pulumi state (ghost resources) |

### Archived (completed phases)

| Document | Contents |
|----------|----------|
| [archive/COMPLETED_POST_LAUNCH.md](./archive/COMPLETED_POST_LAUNCH.md) | All completed work since soft launch (May 11+) |
| [archive/NEXT_STEPS_PRE_LAUNCH.md](./archive/NEXT_STEPS_PRE_LAUNCH.md) | Full build history — 6 sprint priorities, all completed items |
| [archive/STATUS_PRE_LAUNCH.md](./archive/STATUS_PRE_LAUNCH.md) | Feature completion status as of Week 6 |
| [archive/PRE_LAUNCH_AUDIT.md](./archive/PRE_LAUNCH_AUDIT.md) | Security & UX audit — all critical/important items resolved |
| [archive/MVP_SCOPE.md](./archive/MVP_SCOPE.md) | Original MVP scope document |
