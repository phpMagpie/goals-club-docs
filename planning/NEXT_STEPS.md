# Next Steps

Last updated: 2026-04-18

---

## 🎯 Current Sprint: Events, Badges & Social

### 1. Events & Organisers System 🏃 IN PROGRESS
We've created the schema and initial seeders for events with unclaimed organisers.

**Completed:**
- ✅ Migration for unclaimed organisers (`user_id` nullable, `is_claimed` flag)
- ✅ Migration for `event_series` table
- ✅ Added `series_id`, `slug`, `distance_km`, `event_type` to events
- ✅ Seeded 7 organisers: parkrun, Great Run, SuperHalfs, Abbott WMM, T100, London Marathon Events, IRONMAN
- ✅ Seeded 13 event series
- ✅ Seeded 9 sample 2026 events
- ✅ Created EVENTS_SYSTEM.md documentation

**Remaining:**
- ⬜ Test Events & Organisers CRUD in admin (create/edit/delete)
- ⬜ Create web UI for browsing events
- ⬜ Add "I want to do this" interest button
- ⬜ Link events to user goals (event-based goals)
- ⬜ Add more events via admin:
  - ⬜ More Great Run events (Great Scottish Run, Great Birmingham Run, etc.)
  - ⬜ SuperHalfs individual races (Lisbon, Prague, Copenhagen, Cardiff, Valencia)
  - ⬜ Remaining WMM races (Tokyo, Boston)
  - ⬜ Popular UK parkrun locations (as individual events)
  - ⬜ More IRONMAN UK events

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

**Remaining:**
- ⬜ Followers/Following lists (click count to see list)
- ⬜ `activity_reactions` table migration
  - `id`, `activity_id`, `user_id`, `reaction_type_id`, `created_at`
  - Unique constraint on (activity_id, user_id)
- ⬜ `reactToActivity(activityId: ID!, reactionTypeId: ID!)` mutation
- ⬜ `removeReaction(activityId: ID!)` mutation
- ⬜ Update `getActivityFeed` to include activities from followed users
- ⬜ Reaction buttons on activity cards (👏 Cheer, 🙌 High Five, 🤩 Impressed, 💪 Keep Going, ❤️ Love It, ✨ Inspired)
- ⬜ Activity feed filtering (All / Following / My Goals)
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

### 5. Admin Interface 🔧 IN PROGRESS
The admin UI (`goals-club-admin`) has CRUD operations for core models. Tested 2026-04-12.

#### CRUD Status & Test Checklist

| Model | List | View | Create | Edit | Delete | Notes |
|-------|------|------|--------|------|--------|-------|
| Users | ✅ | ✅ | — | ✅ | — | Edit via `adminUpdateUser` resolver. Suspend/unsuspend toggle on show page. Hard delete intentionally omitted (cascade risk). |
| Goals | ✅ | ✅ | — | ✅ | ✅ | Edit/delete via `adminUpdateGoal`/`adminDeleteGoal` resolvers (bypass owner check). User-created via web app. |
| Badges | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested.** Types: system, goal, event, category. Delete uses Mantine modal confirmation. Award Badge action on User show page. |
| Categories | ✅ | ✅ | ✅ | ✅ | ✅ | **Fully tested.** Slug auto-generates from name. Icon is emoji picker. Delete uses Mantine modal confirmation. |
| Events | ✅ | ✅ | ✅ | ✅ | ✅ | Built 2026-04-18, needs testing. `isFeatured` field removed (planned as link table later). |
| Organisers | ✅ | ✅ | ✅ | ✅ | ✅ | Built 2026-04-18, needs testing. |
| Orders | ✅ | ✅ | — | — | — | Read-only + status update. Show page has `updateOrderStatus` dropdown. No admin create/edit/delete needed. |

**Legend:** ✅ = Built & tested | ⬜ = Needs building | — = Not applicable (by design)

#### Completed (2026-04-12)
- ✅ Users list + view with real goals/badges/follower counts (4 new field-level AppSync resolvers on `User` type: `goals`, `badges`, `followerCount`, `followingCount`)
- ✅ Fixed `getUser` resolver — removed hardcoded empty arrays, delegated to field-level resolvers
- ✅ Badges full CRUD — fixed `badgeType` enum values to match DB (`system`, `goal`, `event`, `category`), added show route, Mantine modal for delete confirmation
- ✅ Categories full CRUD — create with auto-slug from name, emoji icon picker (28 curated options), colour picker, display order, active toggle, Mantine modal for delete confirmation
- ✅ All delete confirmations use centered Mantine `Modal` with item name, cancel/delete buttons, and loading state (replaced `window.confirm`)

#### Remaining: Finish CRUD for Events & Organisers

**Events create/edit pages:**
- ⬜ Create `pages/events/create.tsx` — fields: organiserId (select), categoryId (select), name, description, eventDate, registrationCloseDate, locationName, locationAddress, websiteUrl, registrationUrl, imageUrl, maxParticipants, pricePence, isApproved, isFeatured
- ⬜ Create `pages/events/edit.tsx` — same fields, pre-populated from `useShow`
- ⬜ Wire routes in `App.tsx`: `/events/create`, `/events/:id/edit`
- ⬜ Add delete with Mantine modal confirmation on event show page
- ⬜ Organiser select dropdown should load from `listOrganisers` query

**Organisers create/edit pages:**
- ⬜ Create `pages/organisers/create.tsx` — fields: name, description, websiteUrl, logoUrl, contactEmail, isVerified, isActive
- ⬜ Create `pages/organisers/edit.tsx` — same fields, pre-populated from `useShow`
- ⬜ Wire routes in `App.tsx`: `/organisers/create`, `/organisers/:id/edit`
- ⬜ Add delete with Mantine modal confirmation on organiser show page

**Patterns to follow (established in Badges/Categories):**
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
- ⬜ Dashboard with stats (users, goals, activities, badges awarded)
- ⬜ Goal Types management (lookup table CRUD)
- ⬜ Organiser approval workflow
- ⬜ Event approval workflow
- ⬜ Activity moderation (flag/remove inappropriate content)
- ⬜ System settings / feature flags
- ⬜ Featured events (link table with placement context, e.g. homepage, category page)


---

## 📅 Prioritized Roadmap

### Week 1: Finish Events & Badges (Current)

**Day 1-2: Events Infrastructure**
- ⬜ Run migrations and seeders for events
- ⬜ Create GraphQL schema for events
- ⬜ Create basic event resolvers
- ⬜ Test event queries in GraphQL playground

**Day 3-4: Badge Images**
- ⬜ Generate remaining badge images via ChatGPT
- ⬜ Add images to `/public/badges/`
- ⬜ Update database with correct image URLs
- ⬜ Test badge display throughout app

**Day 5: Events UI**
- ⬜ Create Events browse page
- ⬜ Event detail page
- ⬜ "I want to do this" functionality

### Week 2: Social Features

**Following System ✅ COMPLETE**
- ✅ `user_follows` migration + unique constraint
- ✅ `followUser` / `unfollowUser` / `isFollowing` resolvers
- ✅ Follow/unfollow UI on profile pages with optimistic updates
- ✅ Follower/following counts on profiles

**Day 3-4: Reactions System**
- ⬜ Create `activity_reactions` migration (or wire up existing reaction_types)
- ⬜ Create reaction resolvers
- ⬜ Add reaction buttons to activity cards
- ⬜ Show reaction counts and who reacted

**Day 5: Polish & Notifications**
- ⬜ Basic notification system for follows/reactions
- ⬜ Test social features end-to-end
- ⬜ Mobile responsive testing

### Week 3: Username & Profile Completion

**Day 1-2: Username Setup**
- ⬜ Create "Choose Username" modal component
- ⬜ Username availability check endpoint
- ⬜ Wire up modal to show on first login
- ⬜ Update profile to show username

**Day 3-4: Public Profiles**
- ⬜ Profile page for viewing other users (`/profile/:username`)
- ⬜ Show public goals and progress
- ⬜ Show earned badges
- ⬜ Follow button on other users' profiles

**Day 5: Testing & Deploy**
- ⬜ Full flow testing
- ⬜ Deploy to dev environment
- ⬜ User testing with Lynsey

### Week 4: Admin Interface ✅ DONE (2026-04-18)

- ✅ Badges full CRUD tested
- ✅ Categories full CRUD tested
- ✅ User view with real data (goals, badges, follows)
- ✅ User edit — `adminUpdateUser` resolver (bypasses user-scoped `updateUser`)
- ✅ User suspend/unsuspend toggle — `suspended` boolean column + migration
- ✅ Goal edit — `adminUpdateGoal` resolver (bypasses owner check)
- ✅ Goal delete — `adminDeleteGoal` resolver (bypasses owner check)
- ✅ Goal show — `getGoal` resolver created (was missing, causing "Goal not found")
- ✅ Award Badge action on User show page (`awardBadge` admin mutation)
- ✅ Orders view with status update
- ✅ Events full CRUD built (create/edit/delete with Mantine modal — needs testing)
- ✅ Organisers full CRUD built (create/edit/delete with Mantine modal — needs testing)
- ✅ `isFeatured` removed from events (will be a link table with placement context later)
- ✅ Fixed: all boolean `=== 1` comparisons in resolvers replaced with `!!` (Aurora returns true/false not 1/0)
- ✅ All `any` types replaced with shared types across all admin pages

### Week 5: Super Goals (Stretch)

**Day 1-2: Super Goals Structure**
- ⬜ Add `parent_goal_id` column to goals table
- ⬜ Create `super_goal` goal_type
- ⬜ Auto-calculate super goal progress from child goals
- ⬜ Auto-complete super goal when all children complete

**Day 3-4: Seed Super Goals**
- ⬜ Create "Visit All NT Sites (UK)" super goal linking 11 regional goals
- ⬜ Split Wainwrights into 7 regional goals (Northern, North Western, Western, Central, Southern, Eastern, Far Eastern)
- ⬜ Create "Complete All Wainwright Regions" super goal

**Day 5: Super Goals UI**
- ⬜ Display child goals within super goal
- ⬜ Progress tracking across child goals
- ⬜ Test and polish

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
| Events | ✅ | ✅ | ✅ | ✅ | ✅ | Built 2026-04-18, needs testing |
| Organisers | ✅ | ✅ | ✅ | ✅ | ✅ | Built 2026-04-18, needs testing |
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

## ✅ Recently Completed (2026-04-18)

- [x] Data: `followUser` resolver — INSERT … SELECT with ON DUPLICATE KEY; fixed `created_at` ambiguity bug
- [x] Data: `unfollowUser` resolver
- [x] Data: `isFollowing` query resolver
- [x] Data: `followerCount` / `followingCount` field-level resolvers on `User` type
- [x] Data: `updateUserGoal(userGoalId: ID!, input: UpdateUserGoalInput!)` mutation + resolver — updates `visibility`, `customTitle`, `customDescription` (COALESCE so only provided fields change; user-scoped via JOIN)
- [x] Data: `UpdateUserGoalInput` added to GraphQL schema
- [x] Web: `useFollow` hook — `isFollowing` state, optimistic toggle, `isToggling` loading state
- [x] Web: Follow/Unfollow button on public profile pages with optimistic local follower count delta
- [x] Web: `UPDATE_USER_GOAL` mutation in `queries.ts`
- [x] Web: `useUpdateUserGoal` hook
- [x] Web: Quick visibility toggle on My Goals page — globe/lock `ActionIcon` per goal card, toggles PUBLIC ↔ PRIVATE with loading spinner; card link navigation is suppressed via `e.preventDefault()` + `e.stopPropagation()`

- [x] Web: Public profile pages at `/profile/:username` — own or public view based on logged-in user
- [x] Data: `getUserByUsername` resolver + schema field (`@aws_api_key @aws_cognito_user_pools`)
- [x] Web: `usePublicProfile` hook with loading/notFound/error states
- [x] Web: Public profiles show only PUBLIC goals, all badges, follower/following counts
- [x] Web: Share button on own profile copies URL to clipboard
- [x] Web: "User not found" empty state with icon and Explore link
- [x] Admin: Users — suspend/unsuspend toggle on show page (`suspended` boolean column + migration)
- [x] Admin: Users — Award Badge modal on show page (`awardBadge` admin mutation with badge select)
- [x] Admin: Goals — edit page (`adminUpdateGoal` resolver, bypasses owner check; status, visibility, title, target, frequency, allowJoins)
- [x] Admin: Goals — delete with confirmation modal (`adminDeleteGoal` resolver)
- [x] Data: Created `getGoal` resolver (was missing — caused "Goal not found" on show page)
- [x] Data: Created `adminUpdateGoal` resolver (admin bypass of `WHERE user_id = userId` owner check)
- [x] Data: Created `adminDeleteGoal` resolver (admin bypass of owner check)
- [x] Data: Created `adminUpdateUser` resolver (admin update by DB id, not cognito sub)
- [x] Data: Added `suspended: Boolean!` to `User` GraphQL type and `UpdateUserInput`
- [x] Data: Migration `20260418130000-add-suspended-to-users.js` — adds `suspended` column to users table
- [x] Data: Fixed all boolean `=== 1` comparisons across resolvers — replaced with `!!` (Aurora Serverless returns JS `true`/`false`, not `1`/`0`)
- [x] Data: Removed `isFeatured`/`is_featured` from all event resolvers and GraphQL schema (will be a link table with placement context later)
- [x] Shared: Added `suspended` to `User` interface and `UpdateUserInput`

- [x] Admin: Badges full CRUD — list, view, create, edit, delete (tested)
- [x] Admin: Categories full CRUD — list, view, create, edit, delete (tested)
- [x] Admin: Categories create — auto-slug from name, emoji icon picker (28 options), colour picker
- [x] Admin: Delete confirmations use Mantine Modal (replaced window.confirm) on badges + categories
- [x] Admin: Fixed badge type enum — changed from invalid (achievement/milestone/special) to DB values (system/goal/event/category)
- [x] Admin: Added missing badges show route + categories show/create/edit routes in App.tsx
- [x] Data: Created 4 field-level AppSync JS resolvers on User type (goals, badges, followerCount, followingCount)
- [x] Data: Fixed getUser resolver — removed hardcoded empty arrays, field-level resolvers now query user_goals/user_badges/user_follows
- [x] Data: Fixed APPSYNC_JS runtime issue — removed parseFloat (not supported), used UPPER() in SQL instead of JS toUpperCase for enum fields
- [x] Admin: Fixed getUser GraphQL query — changed UserGoal fields from `title` to `customTitle` + `goal { id title }`

## ✅ Previously Completed (2026-03-07 to 2026-03-09)

- [x] Badge system with auto-awarding via pipeline resolvers
- [x] Confetti animation for badge awards
- [x] Badge display on Dashboard, Profile, and Badges pages
- [x] Consolidated badge seeder (24 badges)
- [x] BADGE_DESIGN_REFERENCE.md with kawaii animal mascots
- [x] Gizmo the Pygmy Goat as brand mascot
- [x] Founding member badge criteria and awarding
- [x] Added founding-member.jpg badge image
- [x] Events & Organisers schema (unclaimed organisers support)
- [x] Event series table for grouping related events
- [x] Seeded 7 major event organisers
- [x] EVENTS_SYSTEM.md documentation
- [x] Fixed Goals page RingProgress for recurring goals
- [x] Fixed stray "0" rendering issue (boolean coercion)
- [x] Username system restored (was temporarily removed)
- [x] Profile page retry logic for auth token propagation
- [x] APPSYNC_JS_RUNTIME.md updated with new limitations

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

- Virtual events support
- Training plans linked to events
- Event reviews and ratings
- parkrun location database (2000+ locations)
- Strava/Garmin integration
- Achievement sharing to social media
- Leaderboards (optional, per goal)
- Teams/Groups functionality
- Challenges (time-limited goals)
- Premium features / subscriptions




