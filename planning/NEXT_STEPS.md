# Next Steps

Last updated: 2026-05-16

---

## 📍 Current Phase: Soft Launch (Friends Beta) — Dual Environment

The app is live in both environments:
- **Prod**: `thegoalsclub.co.uk` (public-facing, Strava webhooks active)
- **Dev**: `dev.thegoalsclub.co.uk` (development/testing)
- **Admin**: `admin.thegoalsclub.co.uk` (prod) / `dev-admin.thegoalsclub.co.uk` (dev)

Strava is approved for 999 athletes. See [archive/NEXT_STEPS_PRE_LAUNCH.md](./archive/NEXT_STEPS_PRE_LAUNCH.md) for full build history.

---

## 🔜 Up Next


### CloudWatch Alarms
- DLQ depth > 0 (Strava webhook failures)
- Lambda error rate threshold
- Strava rate limit 80% threshold (EMF metrics already emitted)

---

## ✅ Completed — May 16, 2026

### AWS Budget Alerts
- **Three monthly cost budgets** at $25, $50, $75 thresholds — created in prod stack only (account-scoped)
- Both actual and forecasted spend alerts, emailing `paul@thegoalsclub.co.uk`
- Module: `goals-club-data/packages/infra/modules/cost-monitoring.ts`

### Production Stack Deployment
- **All three repos** (data, web, admin) now deploy to both `dev` and `prod` via `make deploy_dev` / `make deploy_prod`
- **Makefile refactored** in all repos: `_pulumi_setup` → `_pulumi_preview` (generates .env) → `build` → `_pulumi_up` → `invalidate_cache` → `backup_stack`
- **`setup.ts`** reads `.env`/`.env.prod` and `.secrets`/`.secrets.prod` based on `GOALS_CLUB_ENVIRONMENT`, pushes config to Pulumi
- **`initialise_stack.sh`** added to admin repo (data/web already had it)
- **`.gitignore`** updated in all repos to exclude `.env.prod`, `.secrets.*` — caught and unstaged accidentally staged secrets files
- **Cognito `logout_uri`** fix — admin infra now generates `VITE_COGNITO_LOGOUT_URI` dynamically from `adminUrl` (was stale `dev-admin` value)

### Strava Webhook — Prod Only
- **Webhook subscription is now prod-only** — `strava-webhook-subscription.ts` wrapped in `isProduction()` check
- Dev deploys no longer steal the webhook from prod
- Dev can still use Strava OAuth + manual sync; just won't receive automatic activity pushes
- Strava app "Authorization Callback Domain" set to `thegoalsclub.co.uk` (covers all subdomains for OAuth)

### UX Improvements
- **Explore page**: Join goal no longer navigates away — stays on page with filters/search intact, card updates to "Joined" via refetch
- **Explore page**: "Load More" button now visible when "Not joined only" filter is active
- **Explore page**: Past event goals hidden server-side — `listPublicGoals` resolver now LEFT JOINs events table, excludes goals where `event_date < CURDATE()`
- **Error handling**: Auto-retry for RDS cold starts (3 retries: 3s→5s→8s) with friendly beta message if exhausted. New `ErrorAlert` component used across all pages (yellow for wake-up, red for other errors, with "Try Again" button)

### Infrastructure Cost Optimization
- **Removed VPC Endpoint for Secrets Manager** — saved ~$7.30/mo per environment ($14.60/mo total)
- Migration Lambda now receives DB credentials via environment variables (Pulumi secrets) instead of fetching from Secrets Manager at runtime
- Estimated monthly cost reduced from ~$24–62 to ~$10–48 (dev + prod combined)

---

## ✅ Completed — May 11, 2026

### Strava Activity Claiming
- **Claim modal** — users can link a Strava activity to a collection goal item (e.g. a parkrun course). GPS proximity scoring (0-50 pts) + name similarity scoring (0-40 pts), sorted by best match. Search/filter supported.
- **New resolvers**: `listMyStravaActivitiesForItem` (scoring), `claimStravaActivity` (links activity as ITEM_COMPLETION)
- **Migration**: `start_lat`, `start_lng`, `route_polyline` added to `strava_activities`; `strava_activity_id` FK added to `user_goal_activities`
- **Webhook processor** updated to capture GPS start coordinates and route polyline from Strava

### Strava Historical Sync
- **Settings page** "Sync History" button — fetches all past activities from Strava API (paginated at 200/page), dedup-inserts with GPS + polyline
- **Lambda refactored** to router pattern (`event.info.fieldName` switch) handling both `connectStrava` and `syncStravaHistory`
- Lambda timeout increased 30→120s for large history syncs

### Strava Route Maps
- **Inline route map** on claimed items — decodes Google Encoded Polyline and renders on Mapbox GL with Strava orange route line, start/end markers
- New `StravaRouteMap` component + `decodePolyline` utility
- Linked Strava activity details (name, distance, time, link to Strava) shown on completed item cards

### Infrastructure & Types
- GraphQL schema updated: `stravaActivityId` + `stravaActivity` relation on `UserGoalActivity`
- `listUserGoalActivities` resolver updated with LEFT JOIN to `strava_activities`
- `@goals-club/shared` bumped to 0.0.12, published to CodeArtifact
- Settings page layout fixed for mobile (buttons now stack vertically)
- AppSync JS runtime doc updated with new restrictions discovered (no `.sort()` callbacks, no `Math.cos/sqrt`, `util.time.parseFormattedToEpochMilliSeconds` unreliable)

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
| **Notifications** | 2-3 days | DynamoDB table, bell icon, read/unread state. Covers: follows, reactions, badge awards |
| **Mobile polish** | Ongoing | Address any issues testers report on phone viewports |

### Medium-Term (months 2-3)

| Feature | Effort | Notes |
|---------|--------|-------|
| **Super Goals** | 1 week | Parent/child goal hierarchy. `parent_goal_id` column, auto-progress from children. NT regional goals, Wainwright regions |
| **Group Challenges** | 2-3 weeks | B2B2C feature — invite links, leaderboards, time-bounded challenges. See [GROUP_CHALLENGES.md](./GROUP_CHALLENGES.md) |
| **Pipeline resolver conversions** | Ongoing | Convert `joinGoal`, `createGoal`, `followUser` etc. to pipelines when touching them. Tech debt — no user impact |

### Long-Term (months 4+)

| Feature | Notes |
|---------|-------|
| **Garmin integration** | Second fitness platform alongside Strava |
| **Achievement sharing** | Share badges/completions to social media |
| **Premium tier** | Group Challenges Pro (£9.99/mo), analytics dashboard |
| **Public launch** | Marketing site, App Store |
| **Event reviews & ratings** | Community content on events |
| **Training plans** | Linked to events and goals |

---

## 🔧 Tech Debt

| Item | Priority | Notes |
|------|----------|-------|
| Pipeline resolver conversions | Low | 6 candidates identified. Do when touching those resolvers |
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
| [BADGE_DESIGN_REFERENCE.md](./BADGE_DESIGN_REFERENCE.md) | Kawaii animal mascot specs for badge artwork |
| [TECHNICAL_ARCHITECTURE.md](./TECHNICAL_ARCHITECTURE.md) | System architecture overview |
| [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) | Database schema reference |
| [EVENTS_SYSTEM.md](./EVENTS_SYSTEM.md) | Events & organisers system design |
| [SCROLL_RESTORATION.md](./SCROLL_RESTORATION.md) | Scroll restoration implementation notes |
| [APPSYNC_JS_RUNTIME.md](./APPSYNC_JS_RUNTIME.md) | AppSync JS resolver limitations & workarounds |

### Archived (completed phases)

| Document | Contents |
|----------|----------|
| [archive/NEXT_STEPS_PRE_LAUNCH.md](./archive/NEXT_STEPS_PRE_LAUNCH.md) | Full build history — 6 sprint priorities, all completed items |
| [archive/STATUS_PRE_LAUNCH.md](./archive/STATUS_PRE_LAUNCH.md) | Feature completion status as of Week 6 |
| [archive/PRE_LAUNCH_AUDIT.md](./archive/PRE_LAUNCH_AUDIT.md) | Security & UX audit — all critical/important items resolved |
| [archive/MVP_SCOPE.md](./archive/MVP_SCOPE.md) | Original MVP scope document |
