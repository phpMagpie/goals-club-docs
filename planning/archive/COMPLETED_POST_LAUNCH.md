# Completed Work — Post Launch

Archived completed items from [NEXT_STEPS.md](../NEXT_STEPS.md). Covers soft launch period onwards.

---

## ✅ May 18, 2026

### SES Production Access & Email Deliverability
- **SES production access** granted — account moved out of sandbox, can now send to any email address
- **Custom MAIL FROM domain**: `mail.thegoalsclub.co.uk` with MX + SPF records for SPF alignment
- **DMARC record**: `_dmarc.thegoalsclub.co.uk` → `v=DMARC1; p=none; rua=mailto:paul@thegoalsclub.co.uk`
- All three email auth checks now pass: DKIM ✅, SPF ✅, DMARC ✅
- Cognito configured with SES DEVELOPER mode: sends from `The Goals Club <noreply@thegoalsclub.co.uk>`
- Pulumi manages all DNS records (Route53) and SES resources — no manual setup needed

### Strava Webhook Hardening
- Webhook processor now handles both `create` AND `update` events — safe via `INSERT IGNORE` dedup on `strava_activity_id`
- Identified that Strava's "Add People" matched activity feature does NOT send webhooks to tagged users — requires manual Sync History

### Strava Sync Performance
- Removed expensive backfill loop from `syncStravaHistory` — existing activities skip with 1 SQL query instead of N per goal link
- Lambda timeout increased 120s → 300s
- Still times out for large histories (216+ activities) due to AppSync's hard 30s resolver timeout — async sync needed (tracked in NEXT_STEPS)

---

## ✅ May 17, 2026

### Event Goal Completion via Strava Claim
Claiming a Strava activity on an event/milestone goal (no items) now auto-completes the goal. Users can also remove linked activities and re-claim.

**Data backend (goals-club-data):**
- **`claimStravaActivity`** — converted from unit resolver to **pipeline resolver** (2 functions):
  - `verifyAndInsertStravaActivity` — verifies ownership + inserts `user_goal_activities` record
  - `completeGoalForStravaClaim` — updates `user_goals.status` to `COMPLETED` for goal-level claims (no `goalItemId`)
- **`deleteGoalActivity`** — converted from unit resolver to **pipeline resolver** (2 functions):
  - `deleteGoalActivityRecord` — looks up `user_goal_id`, then deletes the activity
  - `resetGoalStatusAfterDelete` — resets goal status to `ACTIVE` if no completion activities remain (handles undo of Strava claim)
- Old unit resolver source files removed; infra auto-skips unit resolvers when pipeline resolver exists for same field

**Web frontend (goals-club-web):**
- **`DELETE_GOAL_ACTIVITY` mutation** added to queries
- **`useDeleteGoalActivity` hook** — wraps the mutation
- **GoalDetail page** — linked Strava activity cards on event goals now have a red trash icon remove button. Removing the last completion resets goal to ACTIVE, re-showing the "Link Strava Activity" button

**Key learnings (documented in APPSYNC_JS_RUNTIME.md):**
- `ctx.prev.result` in a pipeline function's **response** handler is the raw data source result from its own request — NOT the previous function's return value
- Always use `ctx.stash` to pass complex objects between pipeline functions
- Pipeline resolver's response should read from `ctx.stash` as fallback: `ctx.stash.result || ctx.prev.result`
- Pulumi state can get stale when manually deleting AWS resources — use `pulumi state delete <urn>` to clear ghost entries before redeploying

### Group Challenges — Phase 1
Full implementation of the groups system, deployed to both dev and prod. See [GROUP_CHALLENGES_PHASE1.md](../GROUP_CHALLENGES_PHASE1.md) for design spec.

**Data backend (goals-club-data):**
- **3 new tables**: `groups`, `group_members`, `group_goals` (migration deployed)
- **12 unit resolvers**: `getGroup`, `getGroupByInviteCode`, `listMyGroups`, `listGroupMembers`, `listGroupGoals`, `getGroupLeaderboard` (with `startDate`/`endDate` monthly filtering), `getGroupActivityFeed`, `createGoal` (fix: removed non-existent `completed_at` column), `updateGroup`, `deleteGroup`, `leaveGroup`, `removeGroupGoal`, `updateMemberRole`
- **4 pipeline resolvers**: `createGroup` (3 functions), `joinGroup` (3 functions — auto-join removed), `addGroupGoal` (1 function — auto-join removed), `createGoal` (existing, bug fixed)
- **Notifications**: `createGroupJoinNotification` pipeline function — notifies group owner/admins when someone joins
- **Bug fixes**: `util.toJson()` → `JSON.stringify()` in 3 notification pipeline functions (VTL-only API used in JS runtime), removed non-existent `created_at` column from `user_goals` INSERT in two pipeline functions
- **Test data seeder**: `20260517000001-group-demo-data.js` — 7 demo users, 6 months of activity data across 2 goals. Run via `make db_seed_test`, undo via `make db_unseed_test`. Tracked separately in `SequelizeTestSeedMeta`

**Web frontend (goals-club-web):**
- **Groups list page** (`/groups`) — user's groups with create button
- **Create Group page** (`/groups/new`) — name, description, invite code, visibility
- **Group Detail page** (`/groups/:id`) — tabs for goals, leaderboard, feed, members
- **Invite join flow** (`/groups/join/:inviteCode`) — moved to authenticated layout (requires login)
- **Add Goal to Group modal** — two tabs: "Search Existing" (debounced search of public goals) and "Create New" (inline goal creation form with type/frequency/unit)
- **Monthly leaderboard filtering** — month picker dropdown defaulting to current month, last 12 months available
- **Explicit goal joining** — "Join" button on goal cards for members who haven't joined a specific goal, "Joined" badge for those who have (auto-join was removed)
- **`useGroup` hook** — manages group state, members, goals, leaderboard, feed, CRUD operations

**Key design decisions made during implementation:**
- Any user can create a group (no restrictions)
- Groups can have multiple goals (composite unique on `group_id, goal_id`)
- Members explicitly opt into group goals via "Join" button (no auto-join on group join or goal add)
- Monthly leaderboard via query-time date filtering (`startDate`/`endDate` params) rather than per-month group_goals rows
- Leaderboard aggregates from `user_goal_activities` with date range filtering, falls back to `group_goals.start_date/end_date` when no explicit dates provided

### PR Review Fixes (goals-club-data)
- **Secret redaction**: `PULUMI_CONFIG_PASSPHRASE` no longer logged raw in `setup.ts` and `initialise_stack.sh` — presence-only logging
- **AppSync JS runtime**: Fixed `util.toJson()` → `JSON.stringify()` in `createFollowNotification`, `createReactionNotification`, `createBadgeNotifications` pipeline functions

### PR Review Fixes (goals-club-web)
- **Accessibility**: Stat cards (Followers/Following) now use `UnstyledButton` with keyboard/AT support instead of clickable `Paper`
- **`useScrollRestoration`**: `sessionStorage.setItem` wrapped in try/catch for Safari private mode; `behavior: "instant"` → `"auto"`; `window.history.scrollRestoration` guarded with feature detection
- **`useFollowList` pagination**: Proper offset/hasMore/loadMore support instead of single-page fetch; error state exposed; "Load more" button in FollowListModal
- **FollowListModal navigation**: Modal closes when navigating to a profile via `onClose` propagation
- **`graphqlPublicRequest`**: `fetch` wrapped in try/catch, throws `GraphQLNetworkError`
- **Cognito logout URL**: Removed spurious `redirect_uri` and `response_type` params — only `client_id` + `logout_uri`
- **Recurring goal progress**: Non-breaking space (`\u00a0`) between value and unit
- **Lazy Strava route map**: Extracted `LinkedStravaSection` with `useDisclosure` — Mapbox maps only mount on click

### PR Review Fixes (goals-club-admin)
- **Makefile hardened**: Loads env-specific `.env.<env>`/`.secrets.<env>` files; `validate_env` checks correct suffixed files; `deploy_prod`/`deploy_dev` targets
- **Pulumi config keys**: `setup.ts` now injects `env` and `dataEnv` config keys from `GOALS_CLUB_ENVIRONMENT` so prod admin stack doesn't cross-reference dev data stack
- **tsconfig.json**: Added `setup.ts` to `include` (fixes TS1378 top-level await)

---

## ✅ May 16, 2026

### CloudWatch Alarms
- **Strava DLQ depth** alarm — triggers when dead-letter queue has messages (webhook processing failures)
- **Lambda error alarms** — one per critical Lambda (webhook processor, OAuth, token refresh)
- **Strava rate limit 80%** alarm — triggers at 800/1000 daily API calls via EMF metrics
- All alarms notify via SNS → email (`paul@thegoalsclub.co.uk`)
- Module: `goals-club-data/packages/infra/modules/cloudwatch-alarms.ts`

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

## ✅ May 11, 2026

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

