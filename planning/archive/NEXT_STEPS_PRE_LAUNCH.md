# Next Steps

Last updated: 2026-05-16

---

## 🎯 Current Sprint: Pre-Launch Polish

Everything below is ordered by "what would a new user hitting the app notice first."

### ✅ Priority 1: Followers/Following List Modal — DONE
Follower/following counts now open a modal with a paginated user list.

### ✅ Priority 2: Explore Page — Server-Side Pagination & Search — DONE
Explore page now uses server-side `listPublicGoals` resolver with `offset`, `limit`, `search`, `categoryId`. Load More pagination. Empty-state for zero results.

### ✅ Priority 3: Replace `CATEGORIES` Constant with API `listCategories` — DONE
`CreateGoal` page now uses `useCategories()` hook (API-driven with module-level cache). Also fixed a latent bug: was sending category `slug` as `categoryId`, but the backend expects a UUID — categories were silently null on created goals. Now sends `c.id`. Category dropdown also shows emoji icons.

### ✅ Priority 4: Goal Detail — Loading States & Error Handling — DONE
- Skeleton loader replaces bare spinner (shows header, progress bar, activity list structure)
- Error state: large icon, friendly message, "Try Again" button (calls `refetch()`), "Back to My Goals" nav
- Activity history: "Show more" button loads 10 more at a time (was hard-capped at 10 with dead-end "and X more" text)

### ✅ Priority 5: Mobile Responsiveness Pass — DONE
- Goal Detail tabs shortened ("List View" → "List", "Map View" → "Map") + `grow` prop so tabs fill width on mobile
- Tested Dashboard → Explore → Goal Detail → Profile on phone viewport

### ✅ Priority 6: Soft Launch Prep — DONE
- 404 catch-all route added (`NotFound` page with link to Dashboard)
- Onboarding modal now checks `listMyGoals` — existing users on a new device skip onboarding silently
- `useJoinGoal` fixed: was sending `visibility: ""` (invalid enum) — now omits when not set; backend falls back to user's `default_visibility`
- Strava approved for 999 athletes — onboarding Strava step stays enabled

### Deferred (not blocked, just lower priority)
- **Super Goals** — significant schema + UI work, no user demand yet
- **Pipeline resolver conversions** — tech debt, no user-visible impact. Tackle when touching those resolvers
- **Badge images l3–l5** — art asset task, placeholder works. Design session later
- **Strava approval** — ✅ approved for 999 athletes
- **Admin dashboard stats** — nice-to-have, no user impact

### Reference Docs
- [`STRAVA_API_APPROVAL.md`](./STRAVA_API_APPROVAL.md) — approval status and remaining steps
- [`STRAVA_RATE_LIMITS.md`](./STRAVA_RATE_LIMITS.md) — rate limit analysis, all 4 fixes built & deployed
- [`PROD_STACK_DEPLOYMENT.md`](./PROD_STACK_DEPLOYMENT.md) — guide to standing up a production stack alongside dev

---

## ✅ Completed Sprints

**Completed:**
- ✅ Migration for unclaimed organisers (`user_id` nullable, `is_claimed` flag)
- ✅ Migration for `event_series` table
- ✅ Added `series_id`, `slug`, `distance_km`, `event_type` to events
- ✅ Seeded 7 organisers: parkrun, Great Run, SuperHalfs, Abbott WMM, T100, London Marathon Events, IRONMAN
- ✅ Seeded 13 event series + 20+ seeded events
- ✅ Admin CRUD: Events and Organisers (create/edit/delete/approve)
- ✅ Admin: Event approval creates canonical goal via pipeline (`createCanonicalGoalForEvent`)
- ✅ Admin: Category filter excludes Collection Goals and Habit Goals from events
- ✅ Admin: Pagination on all list pages (20 per page, events 15)
- ✅ Web: Events browse page with category/status filter
- ✅ Web: Event detail page with "I'm Doing This" / Shortlist buttons
- ✅ Web: "I'm Doing This" calls `joinGoal` on event's canonical goal — creates `user_goals` record
- ✅ Web: Committed status checked via `user_goals` (not `user_event_interests`)
- ✅ Web: "View Goal" card shown on event detail when committed
- ✅ Web: "View Event" card shown on goal detail for event-based goals
- ✅ Web: Bidirectional navigation between events and their canonical goals
- ✅ Web: Abandoned goals tab on My Goals + Rejoin button (on list and goal detail)
- ✅ Data: `createEvent`, `updateEvent`, `commitToEvent` converted to pipeline resolvers
- ✅ Data: `fetchEventResult` pipeline function for clean event return with JOINs
- ✅ Data: All resolvers use `util.autoUlid()` (was `util.autoId()` — UUID v4)
- ✅ Data: All seeders use `ulid` package (was `uuid`)
- ✅ Created EVENTS_SYSTEM.md documentation


### 2. Badge System 🏅 ✅ LARGELY COMPLETE — l3–l5 images pending

**Completed:**
- ✅ Badge criteria system with auto-awarding
- ✅ Pipeline resolver for `checkAndAwardBadges`
- ✅ Auto-check badges after logging activities and joining goals
- ✅ Confetti animation and toast notifications
- ✅ Badge display on Dashboard, Profile, and Badges pages
- ✅ 67 badges in seeder (60 progression + 6 prestige Goat + 1 Founding Member)
- ✅ BADGE_DESIGN_REFERENCE.md with kawaii animal mascots
- ✅ Gizmo the Pygmy Goat as brand mascot
- ✅ l1 + l2 images live for all 13 animal tracks (real PNGs)
- ✅ placeholder.png in place (optimised) as fallback for l3–l5
- ✅ `badge_type` enum expanded: added `special` + `prestige` via migration `20260510100000-expand-badge-type-enum.js`
- ✅ `location_goal_completed` dead criterion removed from `awardQualifyingBadges.js`
- ✅ `multi_criteria` guard: fails closed for >3 conditions (was silently ignoring extras)

**Remaining:**
- ⬜ Badge images l3, l4, l5 for all 13 tracks (currently fall back to placeholder)
- ⬜ Badge image goat-l6.png (The Goat — prestige max)

### 3. Social Features: Following & Reactions 👥 ✅ LARGELY COMPLETE

**Completed:**
- ✅ `user_follows` table, `followUser` / `unfollowUser` / `isFollowing` resolvers
- ✅ Follow/Unfollow button on public profile pages with optimistic updates
- ✅ Follower/following counts on profile
- ✅ Visibility system: PUBLIC / FRIENDS / PRIVATE (backend-enforced)
- ✅ Activity feed on Dashboard + Feed page (MINE / FOLLOWING / ALL filters)
- ✅ **Reactions fully working** — `addReaction` / `removeReaction` resolvers
- ✅ One reaction per user per activity (DB constraint + optimistic UI switch, not stack)
- ✅ No self-reactions (backend guard + ReactionBar hidden on own activities)
- ✅ `hasReacted` correctly resolved (fixed Cognito sub vs users.id bug in feed SQL)
- ✅ `ReactionBar` component with emoji picker + optimistic toggle
- ✅ `useReactions` hook — switches emoji correctly, strips previous reaction on change
- ✅ `followUser` resolver fixed — split multi-statement string into 2 separate sql args

**Remaining:**
- ✅ Followers/Following lists — modal with paginated user list (Priority 1, 2026-05-10)
- ✅ Notifications for follows and reactions — pipeline resolvers create notifications for follows (`FOLLOW`), reactions (`REACTION`), and badge awards (`BADGE_AWARDED`). Query/read/mark-read resolvers deployed. (2026-05-16)

### 4. User Profile Completion 👤 ✅ COMPLETE
Users need to set a username after registration.

**Completed:**
- ✅ Username column added back to database
- ✅ `useCurrentUser` detects missing username (`needsProfileSetup`)
- ✅ "Choose Username" modal built and shown on first login
- ✅ Username availability check (real-time validation via `checkUsernameAvailable`)
- ✅ Username validation uses shared `usernameSchema` (3-30 chars, alphanumeric + underscore)
- ✅ Public profile pages at `/profile/:username` — own profile if username matches, otherwise public view
- ✅ `getUserByUsername` resolver added to backend
- ✅ Public profiles show only PUBLIC goals (filtered client-side), all badges, follower/following counts
- ✅ "User not found" empty state for unknown usernames
- ✅ Share button on own profile copies profile URL to clipboard

### 5. Admin Interface 🔧 ✅ COMPLETE (2026-04-19)
The admin UI (`goals-club-admin`) has full CRUD for all core models. All tested.

#### CRUD Status & Test Checklist

| Model | List | View | Create | Edit | Delete | Notes |
|-------|------|------|--------|------|--------|-------|
| Users | ✅ | ✅ | — | ✅ | — | Edit via `adminUpdateUser` resolver. Suspend/unsuspend toggle on show page. Hard delete intentionally omitted (cascade risk). |
| Goals | ✅ | ✅ | — | ✅ | ✅ | Edit/delete via `adminUpdateGoal`/`adminDeleteGoal` resolvers (bypass owner check). User-created via web app. |
| Badges | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested.** Types: system, goal, event, category. Delete uses Mantine modal confirmation. Award Badge action on User show page. |
| Categories | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested.** Slug auto-generates from name. Icon is emoji picker. Delete uses Mantine modal confirmation. |
| Events | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested 2026-04-19.** Approval creates canonical goal. Category filter excludes non-event types. |
| Organisers | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested 2026-04-19.** |
| Orders | ✅ | ✅ | — | — | — | Read-only + status update. Show page has `updateOrderStatus` dropdown. No admin create/edit/delete needed. |

**Legend:** ✅ = Built & tested | ⬜ = Needs building | — = Not applicable (by design)

#### Completed (2026-04-12)
- ✅ Users list + view with real goals/badges/follower counts (4 new field-level AppSync resolvers on `User` type: `goals`, `badges`, `followerCount`, `followingCount`)
- ✅ Fixed `getUser` resolver — removed hardcoded empty arrays, delegated to field-level resolvers
- ✅ Badges full CRUD — fixed `badgeType` enum values to match DB (`system`, `goal`, `event`, `category`), added show route, Mantine modal for delete confirmation
- ✅ Categories full CRUD — create with auto-slug from name, emoji icon picker (28 curated options), colour picker, display order, active toggle, Mantine modal for delete confirmation
- ✅ All delete confirmations use centered Mantine `Modal` with item name, cancel/delete buttons, and loading state (replaced `window.confirm`)

#### Remaining: Finish CRUD for Events & Organisers ✅ COMPLETE (2026-04-19)

**Events create/edit pages:**
- ✅ `pages/events/create.tsx` — organiserId (select), categoryId (select), name, description, eventDate, registrationCloseDate, locationName, locationAddress, websiteUrl, registrationUrl, imageUrl, maxParticipants, pricePence, status
- ✅ `pages/events/edit.tsx` — same fields, pre-populated; Approve button on show page
- ✅ Routes wired in `App.tsx`: `/events/create`, `/events/:id/edit`
- ✅ Delete with Mantine modal confirmation on event show page
- ✅ Organiser select dropdown loads from `listOrganisers` query
- ✅ Category dropdown excludes Collection Goals and Habit Goals (event-inappropriate types)

**Organisers create/edit pages:**
- ✅ `pages/organisers/create.tsx` — name, description, websiteUrl, logoUrl, contactEmail, isVerified, isActive
- ✅ `pages/organisers/edit.tsx` — same fields, pre-populated from `useShow`
- ✅ Routes wired in `App.tsx`: `/organisers/create`, `/organisers/:id/edit`
- ✅ Delete with Mantine modal confirmation on organiser show page

Also update the CRUD table above — Events and Organisers are now tested:
- `useCreate`/`useUpdate`/`useDelete` hooks from `@refinedev/core`
- `useForm` from `@mantine/form` with validation
- `useDisclosure` from `@mantine/hooks` for delete modal state
- Back button navigates to list, form cancel navigates to list or show
- Null coalesce optional string fields before sending (`value || null`)
- Resource mutations already configured in `data-provider.ts`

#### Future Admin Enhancements
- ✅ User edit (username, display name, bio, location, privacy settings)
- ✅ User suspend/unsuspend
- ✅ Goal edit (status, visibility, title, target, etc.)
- ✅ Goal delete
- ✅ User Badges management (manual award via Award Badge action)
- ✅ Event approval workflow — Approve button on event show page; triggers `updateEvent` pipeline which calls `createCanonicalGoalForEvent`
- ⬜ Dashboard with stats (users, goals, activities, badges awarded)
- ⬜ Goal Types management (lookup table CRUD)
- ⬜ Organiser approval workflow (no approval mechanism yet — organisers go straight to approved)
- ⬜ Activity moderation (flag/remove inappropriate content)
- ⬜ System settings / feature flags
- ⬜ Featured events (link table with placement context, e.g. homepage, category page)


---

## 📅 Prioritized Roadmap

### Week 1: Events & Badges ✅ COMPLETE

**Events Infrastructure ✅**
- ✅ Migrations and seeders for events, organisers, series
- ✅ GraphQL schema for events, organisers, user_event_interests
- ✅ Event resolvers (getPublicEvent, listPublicEvents, createEvent, updateEvent, commitToEvent)
- ✅ Admin CRUD fully tested

**Badge Images ⬜ (ongoing)**
- ⬜ Generate remaining badge images via ChatGPT
- ⬜ Add images to `/public/badges/`
- ⬜ Update database with correct image URLs

**Events UI ✅**
- ✅ Events browse page with filtering
- ✅ Event detail page with "I'm Doing This" / Shortlist buttons
- ✅ "I'm Doing This" joins canonical goal + bidirectional goal ↔ event navigation

### Week 2: Social Features — Following ✅ COMPLETE, Reactions ⬜ PENDING

**Following System ✅ COMPLETE**
- ✅ `user_follows` migration + unique constraint
- ✅ `followUser` / `unfollowUser` / `isFollowing` resolvers
- ✅ Follow/unfollow UI on profile pages with optimistic updates
- ✅ Follower/following counts on profiles

**Reactions System ✅ COMPLETE (Week 5)**
- ✅ Reaction buttons on activity cards with emoji picker
- ✅ Reaction counts, one-per-user, optimistic toggle

**Notifications ✅ COMPLETE (2026-05-16)**
- ✅ `notifications` table with `type` (`FOLLOW`, `REACTION`, `BADGE_AWARDED`), `title`, `message`, `data` (JSON), `read_at`
- ✅ Pipeline resolvers: `followUser` → `createFollowNotification`, `addReaction` → `createReactionNotification`, `checkAndAwardBadges` → `createBadgeNotifications`
- ✅ Self-notification suppression (follow yourself, react to own activity)
- ✅ Graceful failure — notification insert errors don't fail the parent mutation
- ✅ `listMyNotifications` query (paginated, with `unreadCount`)
- ✅ `unreadNotificationCount` query
- ✅ `markNotificationRead` / `markAllNotificationsRead` mutations
- ✅ Auto-detection of pipeline resolver fields in `appsync-rds.ts` (prevents UNIT/PIPELINE conflicts on deploy)

### Week 3: Username & Profile Completion ✅ COMPLETE

- ✅ "Choose Username" modal on first login
- ✅ Username availability check (`checkUsernameAvailable`)
- ✅ Public profile pages at `/profile/:username`
- ✅ Show public goals, all badges, follower/following counts
- ✅ Follow button on other users' profiles
- ✅ Share button copies profile URL to clipboard

### Week 4: Admin Interface ✅ COMPLETE (2026-04-19)

- ✅ All core models: Users, Goals, Badges, Categories, Events, Organisers, Orders
- ✅ Event + Organiser CRUD fully tested
- ✅ Event approval workflow (creates canonical goal)
- ✅ Pagination on all list pages

### Week 5: Reactions & Activity Feed Polish ✅ COMPLETE (2026-05-10)

**Reactions UI ✅**
- ✅ `addReaction` / `removeReaction` resolvers (fixed Cognito ID bug)
- ✅ One reaction per user per activity (DB constraint + optimistic switch)
- ✅ No self-reactions (backend guard + UI hides picker on own activities)
- ✅ `hasReacted` correctly populated in feed (fixed sub vs users.id)
- ✅ `ReactionBar` component + `useReactions` hook with optimistic toggle
- ✅ Activity feed filtering tabs (ALL / FOLLOWING / MINE)
- ✅ Load More pagination on feed

**Infrastructure ✅**
- ✅ All 5 post-squash migrations folded into single squashed schema
- ✅ Test data moved to `seeders/test/` — excluded from production `db_reset` / `db_seed`
- ✅ `db_seed_test` / `db_unseed_test` use new `seed:test` / `seed:test:undo` actions

### Week 6: Strava Integration ✅ COMPLETE (2026-04-20)

See completed section above and [`STRAVA_API_APPROVAL.md`](./STRAVA_API_APPROVAL.md) for pending approval steps.



### Week 7: Super Goals ⬜ (Moved from Week 5)

**Super Goals Structure**
- ⬜ Add `parent_goal_id` column to goals table
- ⬜ Create `super_goal` goal_type
- ⬜ Auto-calculate super goal progress from child goals
- ⬜ Auto-complete super goal when all children complete

**Seed Super Goals**
- ⬜ Create "Visit All NT Sites (UK)" super goal linking 11 regional goals
- ⬜ Split Wainwrights into 7 regional goals (Northern, North Western, Western, Central, Southern, Eastern, Far Eastern)
- ⬜ Create "Complete All Wainwright Regions" super goal

**Super Goals UI**
- ⬜ Display child goals within super goal
- ⬜ Progress tracking across child goals

---

## 🔧 Technical Debt / Improvements

### `@goals-club/shared` Types & Validation — ✅ Aligned 2026-04-18

All types and validation schemas now match the GraphQL schema exactly. Published as `@goals-club/shared@0.0.6`. Both admin and web wired up to use shared types.

**All done** — `CATEGORIES` constant replaced with `useCategories()` hook (Priority 3, 2026-05-10).

### Admin CRUD — ✅ All Core Models Complete (2026-04-18)

**CRUD test checklist:**

| Resource | List | View | Create | Edit | Delete | Status |
|----------|------|------|--------|------|--------|--------|
| Users | ✅ | ✅ | — | ✅ | — | Edit & suspend built 2026-04-18 |
| Goals | ✅ | ✅ | — | ✅ | ✅ | Edit & delete built 2026-04-18 |
| Badges | ✅ | ✅ | ✅ | ✅ | ✅ | Tested 2026-04-12 |
| Categories | ✅ | ✅ | ✅ | ✅ | ✅ | Tested 2026-04-12 |
| Events | ✅ | ✅ | ✅ | ✅ | ✅ | Tested 2026-04-19 |
| Organisers | ✅ | ✅ | ✅ | ✅ | ✅ | Tested 2026-04-19 |
| Orders | ✅ | ✅ | — | — | — | Untested (has status update) |

### AppSync Resolver Architecture — Establish Convention (Audited 2026-04-12)

Currently 67 unit resolvers and 7 pipeline resolvers (`checkAndAwardBadges`, `logGoalActivity`, `followUser`, `addReaction`, `createEvent`, `updateEvent`, `commitToEvent`). The pipeline resolvers are cleaner and more extendable — they separate concerns into reusable functions and make multi-step flows explicit.

**Convention going forward:**
- **Pipeline resolvers** for any operation that involves multi-step logic, side effects, or orchestration (e.g. log activity → update period → check streaks, or create order → process payment → send notification)
- **Unit resolvers** are acceptable for genuinely simple single-query operations (e.g. `getCategory`, `listReactionTypes`) — wrapping these in a pipeline with one function adds overhead with no benefit
- **Field-level resolvers** (e.g. `User.goals`, `User.badges`) should remain unit resolvers — they're inherently single-query by design

**Candidates for pipeline conversion** (unit resolvers that currently contain logic that would benefit from being split):
- ⬜ `awardBadge` — could chain: validate badge exists → check user doesn't already have it → award
- ⬜ `createGoal` / `createCustomGoal` — could chain: insert goal → auto-join creator → check badge eligibility
- ⬜ `joinGoal` — could chain: insert participant → update participant count → check badge eligibility
- ⬜ `logActivity` (legacy) — similar to `logGoalActivity` pipeline pattern
- ⬜ `updateOrderStatus` — could chain: update status → send notification

**Not worth converting** (simple CRUD, single SQL statement):
- All `list*`, `get*` queries (pure reads)
- All `create*`, `update*`, `delete*` admin CRUD (straightforward insert/update/delete)
- All field-level resolvers (`User.goals`, `User.badges`, etc.)
- Simple mutations (`unfollowUser`, `removeReaction`, `deleteGoalActivity`)

**Reusable function opportunities** — as pipelines grow, extract shared functions:
- `checkBadgeEligibility` — reusable across any mutation that might trigger a badge
- `sendNotification` — reusable across follows, reactions, badge awards, order updates
- `validateOwnership` — reusable auth check (does this user own this resource?)

### Explore Page Pagination — ✅ COMPLETE (Priority 2, 2026-05-10)
- ✅ Server-side pagination via `listPublicGoals` with `offset`, `limit`, `search`, `categoryId`
- ✅ "Load More" pagination
- ✅ Search/filter server-side

### Goal Detail Page Improvements — ✅ COMPLETE (Priority 4, 2026-05-10)
- ✅ Skeleton loader replaces bare spinner
- ✅ Error state with retry button + "Back to My Goals" nav
- ✅ Activity history "Show more" pagination (10 at a time)

### Mobile Responsiveness
- ⬜ Test all pages on mobile viewports
- ⬜ Fix any layout issues
- ⬜ Touch-friendly interactions

### Performance
- ⬜ Add caching for frequently accessed data
- ⬜ Optimize GraphQL queries (reduce over-fetching)
- ⬜ Image optimization for badge assets

---

## 📝 Documentation Updates Needed

- ⬜ Update STATUS.md with current progress
- ⬜ Update MVP_SCOPE.md if priorities have shifted
- ⬜ API documentation for new event endpoints
- ⬜ Update README files with setup instructions

---

## ✅ Recently Completed (2026-05-10)

**Strava rate limit hardening (all 4 fixes deployed):**
- ✅ Webhook returns 500 on Strava 429 (was silently dropping activities — permanent data loss)
- ✅ CloudWatch EMF metrics for rate limit headers (`GoalsClub/Strava` namespace, warns at 80%)
- ✅ SQS decoupling — webhook split into receiver (enqueue) + processor (SQS-triggered, retries on failure, DLQ after 5 attempts)
- ✅ Proactive token refresh Lambda (EventBridge every 5 hours, 6-hour look-ahead)
- ✅ `@aws-sdk/client-sqs` added to infra dependencies; Makefile updated with 2 new build targets
- ✅ See [`STRAVA_RATE_LIMITS.md`](./STRAVA_RATE_LIMITS.md) for full analysis

**Scroll restoration fix:**
- ✅ `useScrollRestoration` — moved save to scroll listener (StrictMode-safe), rAF retry loop for large grids
- ✅ `usePublicGoals` — filter key comparison replaces `skipFirstFetch` flag (StrictMode double-invoke safe)
- ✅ `history.scrollRestoration = "manual"` in `index.tsx`

**National Trust data refresh:**
- ✅ Replaced all legacy NT JSON files with fresh 618-place dataset scraped from nationaltrust.org.uk API
- ✅ Remapped regions from 11 old groups → 10 clean geographic regions
- ✅ Added `transform-national-trust.js` script; removed stale intermediate data files

**Badge system fixes:**
- ✅ 67-badge seeder consolidated; l1 + l2 real PNG images for all 13 tracks
- ✅ `badge_type` enum expanded (`special`, `prestige`) — now in squashed schema
- ✅ Dead `location_goal_completed` criterion removed from `awardQualifyingBadges.js`
- ✅ `multi_criteria` fails closed for >3 conditions (was silently ignoring extras)

**Reactions — full implementation:**
- ✅ `addReaction` resolver fixed: Cognito sub → `users.id` lookup; self-reaction guard
- ✅ `removeReaction` resolver fixed: Cognito sub → JOIN on `users.id`
- ✅ `getActivityFeed` `hasReacted` fixed: was comparing `r.user_id` against Cognito sub (always false)
- ✅ One reaction per user per activity: DB constraint changed + `ON DUPLICATE KEY UPDATE` switch behaviour
- ✅ `ReactionBar` component + `useReactions` optimistic hook (strips previous reaction on switch)
- ✅ Feed hides ReactionBar on own activities (frontend + backend guard)

**Resolver bugs fixed:**
- ✅ `followUser` — multi-statement SQL string split into separate `sql` args (RDS Data API limit of 2)

**Infrastructure:**
- ✅ Migrations squashed to single file (`20260419000001-squashed-initial-schema.js`) — includes Strava tables, badge enum, reactions constraint
- ✅ Test data moved to `seeders/test/` — never runs on production `db_reset` / `db_seed`
- ✅ `seed:test` / `seed:test:undo` Lambda actions with separate `SequelizeTestSeedMeta` tracking

## ✅ Recently Completed (2026-04-21)

- ✅ Web: Privacy policy page (`/privacy`) — covers Strava data, GDPR rights, data sharing, cookies
- ✅ Web: Terms of service page (`/terms`) — covers Strava integration, acceptable use, IP, liability
- ✅ Web: Contact page (`/contact`) — mailto link to support@thegoalsclub.co.uk
- ✅ Web: Privacy, Terms, Contact routes moved inside `AppLayout` (header + footer visible)
- ✅ Web: Footer links (Privacy, Terms, Contact) added to AppLayout — visible on all authenticated pages
- ✅ Web: Goals Club logo added to site header — replaces text-only branding
- ✅ Web: Header scroll-shrink animation (logo 75px → 40px on scroll, title order 3 → 4)
- ✅ Web: "Powered by Strava" attribution added to Goal Detail page (Strava Auto-Sync panel)
- ✅ Web: "Powered by Strava" already on Settings page (Strava icon + text link)
- ✅ Branding: Goals Club logo created (goat + mountain + target + checkmark)
- ✅ Branding: Logo uploaded to Strava API app settings as square app icon
- ✅ Docs: `STRAVA_API_APPROVAL.md` updated with completed items and remaining steps
- ✅ Strava: Approval email sent to Strava with screenshots and app details
- ✅ Docs: Updated STRAVA_API_APPROVAL.md — noted dev mode limits 1 athlete (not 15), marked approval as submitted

## ✅ Recently Completed (2026-04-20)

- ✅ Data: `strava_tokens` table migration
- ✅ Data: `strava_goal_links` table migration
- ✅ Data: Strava OAuth Lambda (outside VPC, RDS Data API) — `connectStrava` mutation
- ✅ Data: `getMyStravaConnection` query resolver (RDS)
- ✅ Data: `disconnectStrava` mutation resolver (RDS)
- ✅ Data: GraphQL schema — `StravaConnection` type, `connectStrava`/`disconnectStrava`/`getMyStravaConnection`
- ✅ Data: `linkStravaActivityType` / `unlinkStravaActivityType` / `listStravaGoalLinks` resolvers
- ✅ Data: Strava webhook Lambda — processes `activity.create`, fetches Strava activity, matches goal links, logs progress
- ✅ Data: Token refresh logic in webhook Lambda (Strava tokens expire every 6 hours)
- ✅ Data: API Gateway HTTP API for webhook (Lambda Function URLs blocked by AWS Org SCP)
- ✅ Data: Strava webhook subscription registered (ID: 341774)
- ✅ Data: Pulumi — `lambda-strava-oauth.ts`, `lambda-strava-webhook.ts` modules
- ✅ Data: Makefile — `strava_oauth_lambda` + `strava_webhook_lambda` build targets
- ✅ Data: `STRAVA_CLIENT_ID` + `STRAVA_CLIENT_SECRET` added to Pulumi config (secrets)
- ✅ Web: Settings page (`/settings`) — Connect/Disconnect Strava
- ✅ Web: Strava callback page (`/callback/strava`)
- ✅ Web: `useStravaConnection` hook
- ✅ Web: `useStravaGoalLinks` hook + `STRAVA_ACTIVITY_TYPES` constant
- ✅ Web: Goal detail — "Strava Auto-Sync" panel with link/unlink UI
- ✅ Web: `GET_MY_STRAVA_CONNECTION`, `CONNECT_STRAVA`, `DISCONNECT_STRAVA` queries in `queries.ts`
- ✅ Web: `VITE_STRAVA_CLIENT_ID` injected via Pulumi web infra
- ✅ Web: Settings link added to Profile page
- ✅ 🎉 **First Strava activity auto-logged to a goal**

## ✅ Recently Completed (2026-04-19)


- ✅ Data: `createEvent` pipeline resolver — `createEventRecord` → `createCanonicalGoalForEvent` → `fetchEventResult`
- ✅ Data: `updateEvent` pipeline resolver — `updateEventRecord` → `createCanonicalGoalForEvent` → `fetchEventResult`
- ✅ Data: `commitToEvent` pipeline resolver — `insertCommitmentRecords` → `getCommitmentResult`
- ✅ Data: `fetchEventResult` pipeline function — always does a fresh SELECT with JOINs for reliable non-null return
- ✅ Data: `createCanonicalGoalForEvent` — creates canonical goal when event approved; no-op if already exists
- ✅ Data: All resolvers migrated from `util.autoId()` to `util.autoUlid()` (ULID format, time-sortable)
- ✅ Data: All seeders migrated from `uuid` package to `ulid` package; `uuid` dependency removed
- ✅ Data: `getPublicEvent` resolver — added `e.goal_id` to SELECT so `goalId` is returned to web
- ✅ Data: `getUserGoal` resolver — added `g.event_id` to SELECT + `eventId` in response mapping
- ✅ Data: Migration `20260419100000-update-category-icons-to-emoji.js` — updated category icons from slugs to emoji
- ✅ Web: Events browse page with category/status filtering
- ✅ Web: Event detail page — "I'm Doing This" (joins canonical goal), Shortlist, Remove buttons
- ✅ Web: `useEventInterest` hook rewired — commits via `joinGoal` (not `commitToEvent`); checks `listMyGoals` for committed status
- ✅ Web: `goalIdRef` pattern to avoid stale closure when event data loads async
- ✅ Web: "View Goal" card on event detail when committed + `userGoalId` resolved from `user_goals`
- ✅ Web: "View Event" card on goal detail for event-linked goals (blue card with `IconCalendar`)
- ✅ Web: `goalId` added to `GET_PUBLIC_EVENT` query and `PublicEvent` interface
- ✅ Web: `eventId` added to `GET_USER_GOAL` query and `Goal` interface
- ✅ Web: Abandoned goals tab on My Goals page; "All" tab excludes abandoned
- ✅ Web: Rejoin button on abandoned goal cards (My Goals list) and goal detail header
- ✅ Web: Navigation order updated — Dashboard → Events → Explore → My Goals → Feed → Badges
- ✅ Admin: Event category dropdown excludes Collection Goals and Habit Goals
- ✅ Admin: Goal edit category dropdown now shows icons
- ✅ Admin: Pagination on all list pages (Goals: 20, Events: 15, Users: 20, Organisers: 20)
- ✅ Admin: `createEvent` and `updateEvent` pipeline resolvers deployed (Pulumi state fixed)
- ✅ Admin: Event approval status correctly sent as `APPROVED` uppercase
- ✅ Data: Fixed `getMe` resolver — self-heals stale `cognito_id` by matching on email (prevents infinite getMe/createUser loop)
- ✅ Data: Fixed `createUser` resolver — uses `ON DUPLICATE KEY UPDATE` to heal `cognito_id` on email match instead of failing on duplicate
- ✅ Data: `User.goals` resolver — filters by visibility (PUBLIC for anyone, FRIENDS for followers, all for owner)
- ✅ Data: `getActivityFeed` resolver — now includes FRIENDS-visibility activities for followed users (was PUBLIC only)
- ✅ Data: `listAllActivities` resolver — fixed `activity_date` serialisation (was appending `.000Z` to date-only values)
- ✅ Data: Added `currentStreak`, `longestStreak`, `periodsCompleted` to `UserGoal` GraphQL type + `User.goals` resolver
- ✅ Data: Test data seeder (`20260420000001-test-user-data.js`) — 8 users, goals, activities, follows, reactions, badges, streak periods
- ✅ Data: `make db_seed_test` / `make db_unseed_test` Makefile targets (rebuild Lambda, deploy, invoke)
- ✅ Data: Lambda deploy now waits for `function-active-v2` before invoking
- ✅ Web: Profile goals show progress bars + streak badges (🏆 best / 🔥 active)
- ✅ Web: Dashboard — Upcoming Events section (featured + 3 cards)
- ✅ Web: Dashboard — Popular Goals section (4 public goals with category + participant count)
- ✅ Web: Dashboard — Recent Activity feed (you + followed users)
- ✅ Web: Profile visibility filter removed (backend now handles it)
- ✅ Admin: Header branding — "🎯 Goals Club" with teal colour + "Admin" suffix
- ✅ Shared: Published 0.0.10 — added `currentStreak`, `longestStreak`, `periodsCompleted` to `UserGoal`
- ✅ All repos: `@goals-club/shared` deduped to root `package.json` only (web + admin)
- ✅ Data: Removed stale `package-lock.json`, added to `.gitignore`

## ✅ Recently Completed (2026-04-18)

- ✅ Data: `followUser` resolver — INSERT … SELECT with ON DUPLICATE KEY; fixed `created_at` ambiguity bug
- ✅ Data: `unfollowUser` resolver
- ✅ Data: `isFollowing` query resolver
- ✅ Data: `followerCount` / `followingCount` field-level resolvers on `User` type
- ✅ Data: `updateUserGoal(userGoalId: ID!, input: UpdateUserGoalInput!)` mutation + resolver — updates `visibility`, `customTitle`, `customDescription` (COALESCE so only provided fields change; user-scoped via JOIN)
- ✅ Data: `UpdateUserGoalInput` added to GraphQL schema
- ✅ Web: `useFollow` hook — `isFollowing` state, optimistic toggle, `isToggling` loading state
- ✅ Web: Follow/Unfollow button on public profile pages with optimistic local follower count delta
- ✅ Web: `UPDATE_USER_GOAL` mutation in `queries.ts`
- ✅ Web: `useUpdateUserGoal` hook
- ✅ Web: Quick visibility toggle on My Goals page — globe/lock `ActionIcon` per goal card, toggles PUBLIC ↔ PRIVATE with loading spinner; card link navigation is suppressed via `e.preventDefault()` + `e.stopPropagation()`

- ✅ Web: Public profile pages at `/profile/:username` — own or public view based on logged-in user
- ✅ Data: `getUserByUsername` resolver + schema field (`@aws_api_key @aws_cognito_user_pools`)
- ✅ Web: `usePublicProfile` hook with loading/notFound/error states
- ✅ Web: Public profiles show only PUBLIC goals, all badges, follower/following counts
- ✅ Web: Share button on own profile copies URL to clipboard
- ✅ Web: "User not found" empty state with icon and Explore link
- ✅ Admin: Users — suspend/unsuspend toggle on show page (`suspended` boolean column + migration)
- ✅ Admin: Users — Award Badge modal on show page (`awardBadge` admin mutation with badge select)
- ✅ Admin: Goals — edit page (`adminUpdateGoal` resolver, bypasses owner check; status, visibility, title, target, frequency, allowJoins)
- ✅ Admin: Goals — delete with confirmation modal (`adminDeleteGoal` resolver)
- ✅ Data: Created `getGoal` resolver (was missing — caused "Goal not found" on show page)
- ✅ Data: Created `adminUpdateGoal` resolver (admin bypass of `WHERE user_id = userId` owner check)
- ✅ Data: Created `adminDeleteGoal` resolver (admin bypass of owner check)
- ✅ Data: Created `adminUpdateUser` resolver (admin update by DB id, not cognito sub)
- ✅ Data: Added `suspended: Boolean!` to `User` GraphQL type and `UpdateUserInput`
- ✅ Data: Migration `20260418130000-add-suspended-to-users.js` — adds `suspended` column to users table
- ✅ Data: Fixed all boolean `=== 1` comparisons across resolvers — replaced with `!!` (Aurora Serverless returns JS `true`/`false`, not `1`/`0`)
- ✅ Data: Removed `isFeatured`/`is_featured` from all event resolvers and GraphQL schema (will be a link table with placement context later)
- ✅ Shared: Added `suspended` to `User` interface and `UpdateUserInput`

- ✅ Admin: Badges full CRUD — list, view, create, edit, delete (tested)
- ✅ Admin: Categories full CRUD — list, view, create, edit, delete (tested)
- ✅ Admin: Categories create — auto-slug from name, emoji icon picker (28 options), colour picker
- ✅ Admin: Delete confirmations use Mantine Modal (replaced window.confirm) on badges + categories
- ✅ Admin: Fixed badge type enum — changed from invalid (achievement/milestone/special) to DB values (system/goal/event/category)
- ✅ Admin: Added missing badges show route + categories show/create/edit routes in App.tsx
- ✅ Data: Created 4 field-level AppSync JS resolvers on User type (goals, badges, followerCount, followingCount)
- ✅ Data: Fixed getUser resolver — removed hardcoded empty arrays, field-level resolvers now query user_goals/user_badges/user_follows
- ✅ Data: Fixed APPSYNC_JS runtime issue — removed parseFloat (not supported), used UPPER() in SQL instead of JS toUpperCase for enum fields
- ✅ Admin: Fixed getUser GraphQL query — changed UserGoal fields from `title` to `customTitle` + `goal { id title }`

## ✅ Previously Completed (2026-03-07 to 2026-03-09)

- ✅ Badge system with auto-awarding via pipeline resolvers
- ✅ Confetti animation for badge awards
- ✅ Badge display on Dashboard, Profile, and Badges pages
- ✅ Consolidated badge seeder (24 badges)
- ✅ BADGE_DESIGN_REFERENCE.md with kawaii animal mascots
- ✅ Gizmo the Pygmy Goat as brand mascot
- ✅ Founding member badge criteria and awarding
- ✅ Added founding-member.jpg badge image
- ✅ Events & Organisers schema (unclaimed organisers support)
- ✅ Event series table for grouping related events
- ✅ Seeded 7 major event organisers
- ✅ EVENTS_SYSTEM.md documentation
- ✅ Fixed Goals page RingProgress for recurring goals
- ✅ Fixed stray "0" rendering issue (boolean coercion)
- ✅ Username system restored (was temporarily removed)
- ✅ Profile page retry logic for auth token propagation
- ✅ APPSYNC_JS_RUNTIME.md updated with new limitations

---

## 🎨 Design Assets Needed

### Badge Images (24 total)
See `BADGE_DESIGN_REFERENCE.md` for full specifications.
- Kawaii/chibi UK countryside animals
- 8 animal mascots across categories
- Gizmo the Pygmy Goat for special badges
- Format: JPG or PNG (SVG preferred but not required)
- Size: 128x128px minimum

### Event/Organiser Logos
- parkrun logo
- Great Run Company logo
- SuperHalfs logo
- Abbott WMM logo
- T100 logo
- London Marathon Events logo
- IRONMAN logo

---

## 💡 Future Ideas (Post-MVP)

### Group Challenges 🏢 (B2B2C Feature)

Enable businesses (e.g. sports massage therapists, gyms, running clubs) to create branded community challenges with invite links, shared goals, and leaderboards.

**Use Case:** A sports massage therapist creates a "Run 100km This Month" challenge, shares an invite link with clients, and tracks their progress on a leaderboard. Prizes for hitting milestones (e.g. "Free massage for anyone who completes 100km").

#### Data Model

```
groups:
  id, name, slug, description, image_url,
  invite_code (unique short code — e.g. "MASSAGE2026"),
  creator_id (FK users), chat_url (WhatsApp/Discord link),
  visibility: PRIVATE | PUBLIC,
  max_members, start_date, end_date,
  created_at, updated_at

group_members:
  id, group_id, user_id,
  role: OWNER | ADMIN | MEMBER,
  joined_at
  unique(group_id, user_id)

group_goals:
  id, group_id, goal_id,
  added_by (FK users),
  leaderboard_enabled: boolean,
  start_date, end_date (time-bounded challenges),
  created_at
  unique(group_id, goal_id)

group_milestones (Phase 2):
  id, group_goal_id,
  title ("Hit 100km"), description,
  target_value, target_unit,
  prize_description ("Free sports massage"),
  badge_id (optional — auto-award on hit),
  created_at
```

#### Join Flow
1. Group admin creates group → gets invite link: `thegoalsclub.co.uk/groups/join/MASSAGE2026`
2. User clicks link → sees group description + goals → "Join Group"
3. Joining auto-joins all group goals (or pick from list)
4. User appears on leaderboard immediately

#### Leaderboard
- No new table — aggregation query on `user_goal_activities` for group members
- `getGroupLeaderboard(groupId, goalId, period: "all" | "month" | "week")`
- Returns: `[{ user, totalValue, currentStreak, rank }]`
- Sortable by: total progress, current streak, longest streak

#### Architecture Notes
- Reuses existing goals, activities, streaks, badges — groups are a social wrapper
- Group activity feed = existing feed filtered by group members
- Invite code flow similar to event "I'm Doing This"
- Group admin is just a user with elevated role in that context
- Privacy: members see each other's progress on group goals only — doesn't expose other goals

#### Key Decisions
- **No built-in chat** — link to WhatsApp/Discord via `chat_url` field
- **Time-bounded challenges** — `start_date`/`end_date` on `group_goals` scopes the leaderboard
- **Multiple admins** — `role: ADMIN` in `group_members`
- **Abuse prevention** — rate-limit group creation, admin can suspend groups

#### Phasing
| Phase | Scope |
|-------|-------|
| **Phase 1** | Groups + members + invite link + group goals + basic leaderboard |
| **Phase 2** | Milestones/prizes + badge awards for group achievements |
| **Phase 3** | Paid groups (Stripe) + group admin analytics dashboard |
| **Phase 4** | Public group directory + featured groups |

#### Revenue Model 💰
- **Free tier:** 1 group, up to 20 members, 3 goals
- **Pro tier (£9.99/mo):** Unlimited groups, custom branding, milestone prizes, analytics
- **Enterprise:** White-label, API access, SSO

### Other Future Ideas
- Virtual events support
- Training plans linked to events
- Event reviews and ratings
- parkrun location database (2000+ locations)
- Garmin integration (alongside Strava)
- Achievement sharing to social media
- Teams/Groups functionality
- Challenges (time-limited goals)
- Premium features / subscriptions

