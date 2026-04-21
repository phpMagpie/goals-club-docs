# Project Status & Roadmap

**Last Updated:** April 21, 2026

---

## 📊 Current Phase: Week 6 — Strava Integration ✅ COMPLETE

### This Week's Focus
- ✅ Strava OAuth connect/disconnect
- ✅ Strava webhook subscription (auto-log activities)
- ✅ Goal ↔ Strava activity type linking
- ✅ First Strava activity auto-logged to a goal

---

## ✅ Completed Features

### Core Infrastructure ✅
- ✅ Aurora Serverless v2 MySQL database
- ✅ AppSync GraphQL API with RDS data source
- ✅ Cognito authentication (Admin + Web clients)
- ✅ S3 + CloudFront deployments (Admin, Web)
- ✅ Route 53 DNS (migrated from Cloudflare)
- ✅ SES for email (DKIM configured)
- ✅ SSL certificate (wildcard *.thegoalsclub.co.uk)

### Admin Console ✅
- ✅ Refine + Mantine 8 setup
- ✅ Full CRUD for Badges, Categories, Events, Organisers
- ✅ Users — list, view, edit (profile fields + privacy), suspend/unsuspend, award badge
- ✅ Goals — list, view, edit (status/visibility/target), delete
- ✅ Goal types management
- ✅ Activity log viewing
- ✅ Orders view with status update

### Web App ✅
- ✅ User authentication (register/login)
- ✅ Dashboard with real data
- ✅ Goals list (my goals) with quick visibility toggle (public/private per goal)
- ✅ Abandoned goals tab + Rejoin functionality
- ✅ Explore page (browse public goals)
- ✅ Join goal functionality
- ✅ Goal detail page with item list
- ✅ Event link on event-based goals ("View Event" card)
- ✅ Events browse page
- ✅ Event detail page with "I'm Doing This" / Shortlist buttons
- ✅ "I'm Doing This" joins the event's canonical goal + shows "View Goal" card
- ✅ Bidirectional navigation: event ↔ goal
- ✅ Public profile pages (`/profile/:username`) — own profile + viewing other users' public goals & badges
- ✅ Username setup modal on first login
- ✅ Follow/unfollow users from public profile pages
- ✅ Follower/following counts on profiles (with optimistic local delta on follow/unfollow)

### Social Features ✅
- ✅ `user_follows` table with unique constraint on (follower_id, following_id)
- ✅ `followUser` / `unfollowUser` mutations and resolvers
- ✅ `isFollowing` query
- ✅ `followerCount` / `followingCount` field-level resolvers on `User` type
- ✅ Follow/Unfollow button on public profiles
- ✅ `useFollow` hook with optimistic updates
- ✅ Visibility system: PUBLIC / FRIENDS / PRIVATE (backend-enforced on User.goals resolver)
- ✅ Activity feed filters by visibility + follower status
- ✅ Dashboard: Recent Activity feed (you + followed users)
- ✅ Dashboard: Upcoming Events (featured + 3 cards)
- ✅ Dashboard: Popular Goals (4 public goals)
- ✅ Profile: Streak badges (🏆 best / 🔥 active) + progress bars on goals
- ✅ `reactions` table + `addReaction`/`removeReaction` resolvers (backend ready)

### Test Data & Developer Experience ✅
- ✅ Test data seeder (8 users, goals, activities, follows, reactions, badges, streaks)
- ✅ `make db_seed_test` / `make db_unseed_test` commands
- ✅ `@goals-club/shared` deduped to root `package.json` only (web + admin)
- ✅ Self-healing auth: `getMe` + `createUser` resolvers auto-fix stale `cognito_id`

### Strava Integration ✅
- ✅ `strava_tokens` table (user_id, access_token, refresh_token, expires_at, strava_athlete_id)
- ✅ `strava_goal_links` table (user_goal_id, strava_activity_type, unique constraint)
- ✅ Strava OAuth Lambda (outside VPC — exchanges code with Strava API, upserts tokens via RDS Data API)
- ✅ AppSync Lambda data source + `connectStrava` mutation resolver
- ✅ `getMyStravaConnection` query + `disconnectStrava` mutation (RDS resolvers)
- ✅ Settings page (`/settings`) — Connect/Disconnect Strava with status display
- ✅ Strava callback page (`/callback/strava`) — handles OAuth redirect, calls `connectStrava`, redirects to profile
- ✅ `useStravaConnection` hook
- ✅ `useStravaGoalLinks` hook — list, link, unlink activity types per goal
- ✅ Goal detail page — "Strava Auto-Sync" panel (visible when Strava connected + recurring goal)
- ✅ Strava webhook Lambda + API Gateway HTTP API (Lambda Function URLs blocked by AWS Org SCP — switched to API Gateway)
- ✅ Webhook processes `activity.create` events — fetches activity from Strava API, matches linked goals, logs progress
- ✅ Token refresh handled in webhook Lambda (6-hour Strava token expiry)
- ✅ Strava webhook subscription registered (ID: 341774)
- ✅ **First Strava activity auto-logged to a goal** 🎉
- ✅ Multiple goals can share same activity type (e.g. "Run" linked to weekly + monthly goals)
- ✅ Multiple activity types can be linked to same goal (e.g. "Run" + "Walk")


- ✅ Wainwrights (214 peaks) seeded
- ✅ National Trust (11 regional goals, 548 sites)
- ✅ Item completion tracking
- ✅ Progress calculation (items completed / total)
- ✅ Multiple visits to same item

### Maps Integration ✅
- ✅ Mapbox GL for location-based goals
- ✅ Conditional display (only for location-based goals)
- ✅ Green markers = completed, Blue = pending
- ✅ Popup details with completion status
- ✅ Auto-fit bounds to show all markers

### Frequency/Distance Goals ✅
- ✅ Target value, unit, frequency fields
- ✅ Period tracking (daily/weekly/monthly)
- ✅ Pipeline resolver for atomic updates
- ✅ `user_goal_periods` table
- ✅ Streak tracking (current + longest streak)
- ✅ Period history UI with visual indicators
- ✅ Example goals: "10,000 Steps a Day", "Cycle to Work 3x per Week"

### Authentication ✅
- ✅ Cognito with Admin/Web app clients
- ✅ Admin users in "Admins" group
- ✅ Self-registration for web users
- ✅ Auto-create user record on first login

---

## 🔜 Next Priorities

### Week 4: Super Goals & Wainwright Regions ⬜ (Moved to Week 7)
- [ ] Add `parent_goal_id` to goals table
- [ ] Create `super_goal` goal_type
- [ ] "Visit All NT Sites (UK)" super goal (links 11 regional goals)
- [ ] Split Wainwrights into 7 regional goals
- [ ] "Complete All Wainwright Regions" super goal
- [ ] Auto-calculate progress from child goals

### Week 5: Badges & Rewards ✅ COMPLETE
- ✅ Badge trigger system (criteria-based checking)
- ✅ Auto-award badges via `checkAndAwardBadges` mutation
- ✅ "Check for Badges" button on Badges page
- ✅ Confetti animation on badge award
- ✅ Modal showing newly earned badges
- ✅ Initial system badges seeded (Founding Member, Goal Getter, First Steps, etc.)
- ✅ Badge display on profile page
- ✅ Auto-check badges after logging activity

### Week 6: Social Features ✅ COMPLETE
- ✅ Activity feed from followed users/goals
- ✅ Follow/unfollow functionality
- ⬜ Reactions/cheers on activities (backend ready, UI pending)
- ✅ Public profile pages

---

## 🗓️ Roadmap

### MVP (Launch Target)
| Week | Focus | Status |
|------|-------|--------|
| 1 | Foundation & Core Goal Flow | ✅ Complete |
| 2 | Activity Logging & Progress | ✅ Complete |
| 3 | Goal Types & Multiple Visits | ✅ Complete |
| **4** | **Admin, Social, Dashboard** | ✅ Complete |
| **5** | **Reactions & Activity Feed Polish** | 🔄 Next up |
| **6** | **Strava Integration** | ✅ Complete |
| 7 | Super Goals & Wainwright Regions | ⬜ Planned |
| 8 | Profile Polish & Launch Prep | ⬜ Planned |

### Post-MVP (After Launch)
| Phase | Focus | Notes |
|-------|-------|-------|
| 2.1 | Physical Merchandise Shop | Need merch partners first |
| 2.2 | Event Organisers | Organiser registration + events |
| 2.3 | Strava API Approval | Requires privacy policy, "Powered by Strava" badge, Strava app settings — see STRAVA_API_APPROVAL.md |
| 2.4 | Local Goal Clubs | Organic community growth needed |

---

## 📁 Project Structure

### Repositories
```
goals-club-data/    # API, Database, Infrastructure
goals-club-admin/   # Admin Console (Refine + Mantine)
goals-club-web/     # Public Web App (Mantine)
goals-club-shared/  # Shared types, helpers, constants
```

### Key Directories (goals-club-data)
```
packages/infra/modules/appsync-rds/
├── schema.graphql              # GraphQL schema
├── resolvers/src/              # Standard resolvers
│   ├── activities/
│   ├── badges/
│   ├── categories/
│   ├── goals/
│   └── users/
├── pipeline-functions/src/     # Pipeline functions
│   ├── insertActivity.js
│   ├── getGoalPeriodInfo.js
│   └── upsertGoalPeriod.js
└── pipeline-resolvers/src/     # Pipeline resolver definitions
    └── logGoalActivity.js
```

---

## 🔧 Technical Notes

### AppSync JS Runtime
**📖 Full guide:** [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md)

Key constraints:
- No `for` loops (use `map`, `forEach`)
- No `++`/`--` operators (use `+= 1`)
- Max 2 SQL statements per resolver (use pipeline resolvers for more)
- MySQL DATETIME format: `YYYY-MM-DD HH:MM:SS`

### Cost Management
- Stacks up: ~$0.30-$5/day
- Stacks down: ~$0.50/month (Route 53 hosted zone only)
- Use `pulumi down` or `pulumi destroy` when not developing
- Aurora auto-pauses after 5 minutes idle

**Idle costs breakdown:**
| Service | Monthly Cost | Notes |
|---------|-------------|-------|
| Route 53 hosted zone | $0.50 | Unavoidable while using AWS DNS — created manually, not managed by Pulumi |
| Secrets Manager | $0.40/secret | Orphaned RDS secrets can linger after destroy — see cleanup below |

**After `pulumi destroy`, always clean up orphaned RDS secrets:**
```bash
AWS_PROFILE=goalsclub aws secretsmanager list-secrets --region eu-west-1 \
  --query "SecretList[*].Name" --output text | tr '\t' '\n' | while read secret; do
  AWS_PROFILE=goalsclub aws secretsmanager delete-secret \
    --secret-id "$secret" --force-delete-without-recovery --region eu-west-1
done
```
Or use the Makefile target: `make cleanup_secrets`

**Actual costs observed:**
| Month | Cost | Notes |
|-------|------|-------|
| March 2026 | $10.31 | Stacks active, multiple destroy/redeploy cycles |
| April 2026 | ~$0.50 | After secret cleanup (Route 53 only) |

### Admin Users
- paul@thegoalsclub.co.uk (Admin)
- lynsey@thegoalsclub.co.uk (Admin)

---

## 🐛 Known Issues

### Build Issue (esbuild)
Occasionally `bun run build` fails with "The service was stopped":

```bash
pkill -9 -f esbuild
pkill -9 -f vite
rm -rf node_modules/.cache node_modules/.vite
bun run build
```

Or use `npx vite build` directly.

---

## 📝 Quick Commands

```bash
# Start dev servers
cd goals-club-web/packages/ui && bun run dev   # Web: http://localhost:5176
cd goals-club-admin/packages/ui && bun run dev # Admin: http://localhost:5175

# Deploy stacks
cd goals-club-data && make deploy
cd goals-club-web && make deploy
cd goals-club-admin && make deploy

# Destroy stacks (save costs)
cd goals-club-data && make down
```

---

## 📚 Documentation Index

| Document | Purpose |
|----------|---------|
| [`PROJECT.md`](./PROJECT.md) | Vision, features, revenue model |
| [`TECHNICAL_ARCHITECTURE.md`](./TECHNICAL_ARCHITECTURE.md) | Infrastructure patterns |
| [`DATABASE_SCHEMA.md`](./DATABASE_SCHEMA.md) | Full schema reference |
| [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md) | Resolver guidelines |
| [`MAP_VIEW_SETUP.md`](./MAP_VIEW_SETUP.md) | Mapbox configuration |
| [`STRAVA_API_APPROVAL.md`](./STRAVA_API_APPROVAL.md) | Strava API approval checklist |

---

## 🏔️ Test Data to Re-seed on Stack Rebuild

| Goal Item | Date | Notes |
|-----------|------|-------|
| Catbells | 12/04/2023 | Really popular, wore brand new Van boots, bad mistake. |

### Admin user accounts after stack rebuild
After `pulumi up`, **do not seed user records manually** — just log in via the web app. The auto-create flow creates the DB record with the correct Cognito `cognito_id` from the JWT automatically.

If you need to pre-assign goals/badges to an account, get the Cognito sub first:
```bash
AWS_PROFILE=goalsclub aws cognito-idp list-users \
  --user-pool-id $(cat .env | grep COGNITO_USER_POOL_ID | cut -d= -f2) \
  --region eu-west-1 \
  --filter "email = \"paul@thegoalsclub.co.uk\"" \
  --query "Users[0].Username" --output text
```
Use that value as `cognito_id` in any seeders that create user rows.

**Self-healing:** The `getMe` resolver also self-heals mismatched `cognito_id`s — if a user row exists with the right email but wrong/null `cognito_id`, it fixes it automatically on first login.

---

## ✅ Completed Log

| Date | Milestone |
|------|-----------|
| 2026-03-01 | Git repos initialized |
| 2026-03-02 | Route 53 DNS migration, Admin user auto-creation |
| 2026-03-03 | Resolver reorganization, First activity logged |
| 2026-03-04 | National Trust 548 locations seeded, Map view added |
| 2026-03-07 | Frequency goals, Streak tracking, Pipeline resolvers |
| 2026-03-08 | Documentation consolidation |
| 2026-03-08 | **Badge system complete** - Auto-award, confetti, modal celebration |
| 2026-04-18 | **Social features complete** - Follow/unfollow, public profiles, follower counts |
| 2026-04-18 | **Goal visibility** - `updateUserGoal` mutation + quick toggle on My Goals page |
| 2026-04-20 | **Strava integration complete** — OAuth, webhook, goal linking, first activity auto-logged |
| 2026-04-20 | **API Gateway webhook** — Lambda Function URLs blocked by AWS Org SCP; switched to API Gateway HTTP API |
| 2026-04-20 | **Strava webhook subscription registered** (ID: 341774) |
| 2026-04-19 | **Events system complete** - Browse, detail, "I'm Doing This" joins canonical event goal |
| 2026-04-19 | **Event ↔ Goal linking** - Bidirectional navigation between events and goals |
| 2026-04-19 | **Abandoned goals** - Abandoned tab on My Goals + Rejoin button |
| 2026-04-19 | **ID consistency** - All resolvers use `util.autoUlid()`, seeders use `ulid` package |
| 2026-04-19 | **Admin pagination** - All list pages paginated (20 per page, events 15) |
| 2026-04-19 | **Admin pipeline resolvers** - `createEvent`, `updateEvent`, `commitToEvent` converted to pipelines with `fetchEventResult` for clean return |
| 2026-04-19 | **Visibility system** - Backend-enforced PUBLIC/FRIENDS/PRIVATE on User.goals + activity feed |
| 2026-04-19 | **Dashboard enrichment** - Upcoming Events, Popular Goals, Recent Activity sections |
| 2026-04-19 | **Profile streaks** - 🏆 best / 🔥 active streak badges + progress bars on profile goals |
| 2026-04-19 | **Test data seeder** - 8 users with goals, activities, follows, reactions, badges, streak periods |
| 2026-04-19 | **Auth self-healing** - getMe/createUser resolvers auto-fix stale cognito_id (prevents infinite loop) |
| 2026-04-19 | **Package dedup** - @goals-club/shared moved to root package.json only (web + admin) |

---

*Document consolidated: March 8, 2026*
*Original sources: PROJECT_STATUS.md, NEXT_STEPS.md*

