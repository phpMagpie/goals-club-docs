# Next Steps

Last updated: 2026-04-19

---

## 🎯 Current Sprint: Events, Badges & Social

### 1. Events & Organisers System 🏃 ✅ COMPLETE

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


### 2. Badge System 🏅 IN PROGRESS
Badge awarding is working. Now need to complete the visual design.

**Completed:**
- ✅ Badge criteria system with auto-awarding
- ✅ Pipeline resolver for `checkAndAwardBadges`
- ✅ Auto-check badges after logging activities and joining goals
- ✅ Confetti animation and toast notifications
- ✅ Badge display on Dashboard, Profile, and Badges pages
- ✅ Consolidated 24 badges in seeder
- ✅ BADGE_DESIGN_REFERENCE.md with kawaii animal mascots
- ✅ Gizmo the Pygmy Goat as brand mascot
- ✅ Created `/public/badges/` folder structure
- ✅ Added founding-member.jpg badge image

**Remaining:**
- ⬜ Update database with correct image extension: 
  ```sql
  UPDATE badges SET image_url = '/badges/founding-member.jpg' WHERE name = 'Founding Member';
  ```
- ⬜ Create/add remaining 23 badge images:
  - ⬜ Goal Joining (Squirrel): first-steps, high-five, ambitious
  - ⬜ Activity Logging (Hedgehog): getting-started, active, committed, century-club
  - ⬜ Goal Completion (Fox): completionist, goal-crusher, perfect-ten
  - ⬜ Exploration (Owl): explorer
  - ⬜ Streaks (Robin): dedicated, weekly-wonder, monthly-master
  - ⬜ Super Goals (Gizmo): super-achiever
  - ⬜ Social (Border Collie): cheerleader, super-supporter
  - ⬜ Wainwrights (Mountain Goat): wainwright-explorer, wainwright-enthusiast, wainwright-master
  - ⬜ National Trust (Sheep): nt-visitor, nt-explorer, nt-completionist
- ⬜ Update seeder with correct file extensions for each badge
- ⬜ Test badge display with real images

### 3. Social Features: Following & Reactions 👥 PARTIALLY COMPLETE

Enable users to follow each other and encourage/react to activities.

**Completed:**
- ✅ `user_follows` table migration (`id`, `follower_id`, `following_id`, `created_at`, unique constraint)
- ✅ `followUser(userId: ID!)` mutation + resolver (INSERT … SELECT with ON DUPLICATE KEY)
- ✅ `unfollowUser(userId: ID!)` mutation + resolver
- ✅ `isFollowing(userId: ID!)` query + resolver
- ✅ `followerCount` / `followingCount` field-level resolvers on `User` type
- ✅ Follow/Unfollow button on public profile pages (`useFollow` hook with optimistic updates)
- ✅ Follower/following counts on profile (with optimistic local delta while toggling)
- ✅ Fixed SQL ambiguity bug in `followUser` resolver (`ON DUPLICATE KEY UPDATE created_at = user_follows.created_at`)
- ✅ Visibility system working: PUBLIC (anyone), FRIENDS (followers only), PRIVATE (owner only)
- ✅ `User.goals` resolver filters by visibility + follower status (backend-enforced)
- ✅ Activity feed shows FRIENDS-visibility content to followers
- ✅ `reactions` table exists with `addReaction`/`removeReaction` resolvers (backend ready)
- ✅ Activity feed on Dashboard (Recent Activity section — you + followed users)

**Remaining:**
- ⬜ Followers/Following lists (click count to see list)
- ⬜ Reaction buttons on activity cards (👏 Cheer, 🙌 High Five, 🤩 Impressed, 💪 Keep Going, ❤️ Love It, ✨ Inspired)
- ⬜ Show reaction counts on activity cards
- ⬜ Notifications for follows and reactions

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

**Reactions System ⬜ (Week 5 — next up)**
- ⬜ Add reaction buttons to activity cards (👏 Cheer, 🙌 High Five, etc.) — backend resolvers already exist
- ⬜ Show reaction counts and who reacted

**Notifications ⬜**
- ⬜ Basic notification system for follows/reactions
- ⬜ Mobile responsive testing

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

### Week 5: Reactions & Activity Feed Polish ⬜ (Next up)

**Reactions UI (backend ready)**
- ⬜ Reaction buttons on activity cards (👏 🙌 🤩 💪 ❤️ ✨)
- ⬜ Show reaction counts + who reacted (tooltip)
- ⬜ `useReaction` hook with optimistic updates
- ⬜ Reactions on Dashboard activity feed + Feed page + Profile activity

**Activity Feed Improvements**
- ⬜ Feed page filtering tabs (All / Following / Mine)
- ⬜ Infinite scroll or "Load More" on feed
- ⬜ Activity cards show goal progress context (e.g. "12/214 Wainwrights")

**Followers/Following Lists**
- ⬜ Click follower/following count to see user list modal
- ⬜ Follow/unfollow from the list

### Week 6: Strava Integration ⬜

**OAuth & Account Linking**
- ⬜ Strava OAuth2 flow (requires live site URL for callback)
- ⬜ `strava_tokens` table (user_id, access_token, refresh_token, expires_at, athlete_id)
- ⬜ Settings page: "Connect Strava" button + connection status
- ⬜ Token refresh Lambda (Strava tokens expire every 6 hours)

**Activity Sync**
- ⬜ Strava webhook subscription (receives new activities in real-time)
- ⬜ Lambda to process Strava webhook → match to user goals → auto-log activity
- ⬜ Activity type mapping: Strava Run → distance goals, Strava Hike → Wainwright check-in, etc.
- ⬜ "Imported from Strava" badge on activity cards
- ⬜ Manual sync button ("Sync Recent Activities")
- ⬜ Deduplication: don't double-count manually logged + Strava-synced activities

**Goal Matching**
- ⬜ Auto-match Strava activities to joined goals by type (run → running goals, ride → cycling goals)
- ⬜ Distance/elevation extraction for progress updates
- ⬜ Location matching for collection goals (Wainwright summit proximity check)

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

**Still to do:**
- ⬜ Replace `CATEGORIES` constant usage in web `CreateGoal` page with API-driven `listCategories` query

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

Currently 67 unit resolvers and 2 pipeline resolvers (`checkAndAwardBadges`, `logGoalActivity`). The pipeline resolvers are cleaner and more extendable — they separate concerns into reusable functions and make multi-step flows explicit.

**Convention going forward:**
- **Pipeline resolvers** for any operation that involves multi-step logic, side effects, or orchestration (e.g. log activity → update period → check streaks, or create order → process payment → send notification)
- **Unit resolvers** are acceptable for genuinely simple single-query operations (e.g. `getCategory`, `listReactionTypes`) — wrapping these in a pipeline with one function adds overhead with no benefit
- **Field-level resolvers** (e.g. `User.goals`, `User.badges`) should remain unit resolvers — they're inherently single-query by design

**Candidates for pipeline conversion** (unit resolvers that currently contain logic that would benefit from being split):
- ⬜ `awardBadge` — could chain: validate badge exists → check user doesn't already have it → award
- ⬜ `createGoal` / `createCustomGoal` — could chain: insert goal → auto-join creator → check badge eligibility
- ⬜ `joinGoal` — could chain: insert participant → update participant count → check badge eligibility
- ⬜ `logActivity` (legacy) — similar to `logGoalActivity` pipeline pattern
- ⬜ `updateOrderStatus` — could chain: update status → send notification (when notifications exist)
- ⬜ `followUser` — could chain: insert follow → send notification (when notifications exist)

**Not worth converting** (simple CRUD, single SQL statement):
- All `list*`, `get*` queries (pure reads)
- All `create*`, `update*`, `delete*` admin CRUD (straightforward insert/update/delete)
- All field-level resolvers (`User.goals`, `User.badges`, etc.)
- Simple mutations (`unfollowUser`, `removeReaction`, `deleteGoalActivity`)

**Reusable function opportunities** — as pipelines grow, extract shared functions:
- `checkBadgeEligibility` — reusable across any mutation that might trigger a badge
- `sendNotification` — reusable across follows, reactions, badge awards, order updates
- `validateOwnership` — reusable auth check (does this user own this resource?)

### Explore Page Pagination
- ⬜ Server-side pagination for public goals (currently client-side filtering)
- ⬜ "Load More" or infinite scroll
- ⬜ Move search/filter to server-side

### Goal Detail Page Improvements
- ⬜ Better loading states
- ⬜ Error handling improvements
- ⬜ Activity history pagination

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

