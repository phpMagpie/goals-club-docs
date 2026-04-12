# Next Steps

Last updated: 2026-03-07

---

## 🎯 Immediate Priority Tasks

### 1. Map View Conditional Display ✅ COMPLETE (2026-03-07)
- ✅ Only show Map tab for goals with location-based items
- ✅ Add `hasLocationData` computed field to check if any items have lat/lng
- ✅ Hide Map tab for non-location goals (e.g., habit goals, distance goals)

### 2. Multiple Visits to Same Item ✅ COMPLETE (2026-03-07)
- ✅ Allow logging visits to the same item more than once
- ✅ First visit with `is_completion = true` marks the item as completed
- ✅ Subsequent visits logged as `is_completion = false` (additional visits)
- ✅ Update UI to show:
  - ✅ Completed (first visit date)
  - 📍 Additional visits (count + dates)
- ✅ Update activity feed to show all visits

### 3. Different Goal Types Implementation ✅ COMPLETE (2026-03-07)

**A. Distance/Time Goals (e.g., "Run 5k/10k/25k each week")** ✅ COMPLETE
- ✅ Add `target_value`, `target_unit`, `target_period` fields to goals
- ✅ Create distance/time goal types with:
  - Target: configurable (5, 10, 25, 50, 10000, etc.)
  - Unit: km, miles, steps, times (all supported)
  - Period: day, week, month (via target_frequency)
- ✅ Activity logging: enter value with unit (distance/time/count)
- ✅ Progress: accumulated value vs target per period
- ✅ Reset progress at period boundary (tracked via user_goal_periods)
- ✅ Example goals seeded:
  - ✅ "10,000 Steps a Day" (target_value: 10000, target_unit: steps, target_frequency: DAILY)
  - ✅ "Cycle to Work 3x per Week" (target_value: 3, target_unit: times, target_frequency: WEEKLY)
- **Note:** Distance/Time and Frequency goals use the same underlying mechanism - accumulate values against a target per period

**B. Frequency Goals (e.g., "Walk the dog 2x per day")** ✅ COMPLETE (2026-03-07)
- ✅ Add `target_frequency`, `frequency_period` fields
- ✅ Create `frequency_goal` goal_type
- ✅ Activity logging: simple check-in (no specific item)
- ✅ Progress: count vs target per period
- ✅ Example goals seeded:
  - ✅ "Cycle to Work 3x per Week"
  - ✅ "10,000 Steps a Day" (frequency-based counting)
- ✅ **Streak Tracking Implementation** ✅ COMPLETE:
  - ✅ Created `user_goal_periods` table to track period completions
  - ✅ Created `listUserGoalPeriods` query resolver
  - ✅ Created `useUserGoalPeriods` hook and UI components
  - ✅ **Pipeline Resolver Implementation:**
    - Created pipeline resolver with 3 functions: `insertActivity`, `getGoalPeriodInfo`, `upsertGoalPeriod`
    - Fixed MySQL datetime formatting (use `YYYY-MM-DD HH:MM:SS` not ISO format)
    - Added to Data Makefile: `build-pipeline-functions` and `build-pipeline-resolvers` targets
  - ✅ Period History UI: shows boxes with date (d/m) + check/X icon per period
  - ✅ Tooltip shows full date range, achieved vs target, and status
  - ✅ Current streak calculation and display
  - ✅ Longest streak calculation and display
  - ✅ Dynamic labels based on frequency (days/weeks/months)

**C. Collection Goals (existing - Wainwrights, NT)** ✅ COMPLETE
- ✅ Already implemented with goal_items
- ✅ Map view for location-based collections
- ✅ Support both location and non-location collections (hasLocationData check)

### 4. Super Goals (Meta Goals)
- ⬜ Create `super_goal` goal_type where items are other goals
- ⬜ Add `parent_goal_id` to goals table for hierarchy
- ⬜ Progress: count completed child goals vs total
- ⬜ Auto-complete super goal when all child goals complete
- ⬜ Example super goals:
  - ⬜ "Visit All National Trust Sites (UK)" - parent of 11 regional NT goals
    - **Note:** Current NT data is from OSM (buildings/infrastructure). Will need manual curation of official NT visitor properties before production.
  - ⬜ "Complete All Wainwright Regions" - split 214 peaks into geographic groups:
    - Northern Fells (24 peaks)
    - North Western Fells (29 peaks)
    - Western Fells (33 peaks)
    - Central Fells (27 peaks)
    - Southern Fells (30 peaks)
    - Eastern Fells (35 peaks)
    - Far Eastern Fells (36 peaks)

### 5. Virtual Badges (After Goal Types Complete)
- ⬜ Auto-award badges based on goal completion triggers
- ⬜ Badge types:
  - **Founding Member** - registered during beta
  - **First Steps** - joined first goal
  - **Getting Started** - logged first activity
  - **Completionist** - completed first goal
  - **Explorer** - completed first location-based goal
  - **Regular** - maintained a weekly goal for 4+ weeks
  - **Dedicated** - maintained a daily goal for 7+ days
  - **Super Achiever** - completed a super goal
  - Goal-specific badges (Wainwright Master, NT Explorer, etc.)

---

## 📅 Prioritized Roadmap (Few Hours/Day)

### Week 1: Foundation & Core Goal Flow

**Day 1-2: Database & Resolvers for Collection Goals**
- ✅ Create new migration with updated schema (goals, goal_items, user_goals, user_goal_activities)
- ✅ Add goal_types lookup table and seeder
- ✅ Update DATABASE_SCHEMA.md documentation
- ✅ Deploy migration to dev environment (`make deploy` in goals-club-data)
- ✅ Update GraphQL schema to match new tables
- ✅ Create resolvers for new tables (goal_types, goal_items, user_goals, user_goal_activities)

**Day 3-4: Seed Data & Join Goal Flow** ✅ COMPLETE
- ✅ Seed Wainwrights data (214 peaks) - from https://github.com/thomaswilsonxyz/wainwright-peaks/blob/main/wainwrights.json
- ✅ Create "Summit all Wainwrights" public goal with 214 goal_items
- ✅ Implement `joinGoal` mutation (creates `user_goals` entry)
- ✅ Fix `listAllGoals` resolver (admin) - proper nested goalType, creator, uppercase enums
- ✅ Fix `listMyGoals` resolver (web) - proper nested goal object, uppercase enums
- ✅ Update web Goals page to handle UserGoal structure
- ✅ Implement `listPublicGoals` resolver properly
- ✅ Fix enum case consistency (store uppercase in DB)
- ✅ Seed National Trust locations (548 sites across 11 UK regions) - 2026-03-04
- ✅ Create 11 regional NT goals (East Midlands 52, East of England 15, North East 25, North West 72, Northern Ireland 10, Scotland 15, South East 120, South West 157, Wales 38, West Midlands 34, Yorkshire 10)

**Day 5: Web App - Browse & Join Goals**
- ✅ Create "Explore Goals" page showing public goals
- ✅ Add "Join Goal" button functionality
- ✅ Update Goals page to show joined goals

### Week 2: Activity Logging & Progress

**Day 1-2: Log Activities**
- ✅ Implement `logGoalActivity` mutation
- ✅ Create "Log Activity" form in web app
- ✅ Show goal items as a checklist (ticked/unticked)
- ✅ Calculate and display progress percentage

**Day 3-4: Goal Detail Page**
- ✅ Create Goal detail page showing all items
- ✅ Show which items are completed vs pending
- ✅ Show activity history for each item
- ✅ Add map view for location-based goals (Wainwrights, NT) - 2026-03-04
  - Created GoalItemsMap component using Mapbox GL
  - Added tab view (List/Map) on Goal Detail page
  - Shows completed (green) vs pending (blue) markers
  - Auto-fits bounds to show all locations
  - Includes legend and progress stats overlay

**Day 5: Testing & Polish** ✅ COMPLETE
- ✅ Test full flow: browse → join → log → complete (Catbells summit logged 12/04/2023)
- ✅ Fix any bugs found
- ✅ Deploy to dev environment

### Week 3: Goal Types & Multiple Visits

**Day 1-2: Map View & Multiple Visits** ✅ COMPLETE (2026-03-07)
- ✅ Only show Map tab for location-based goals
- ✅ Allow multiple visits to same item (completion + additional visits)
- ✅ Update Goal Detail UI to show visit history per item
- ✅ Update activity feed to show all visits

**Day 3-4: Frequency Goals & Streak Tracking** ✅ COMPLETE (2026-03-07)
- ✅ Implement pipeline resolvers for `logGoalActivity`:
  - `insertActivity` - inserts the activity record
  - `getGoalPeriodInfo` - gets goal period info (weekly/daily/monthly)
  - `upsertGoalPeriod` - creates/updates period tracking record
- ✅ Create `user_goal_periods` table for tracking period completions
- ✅ Create Period History UI with visual week boxes (date + ✓/✗)
- ✅ Show streak statistics (current streak, longest streak, periods completed)
- ✅ Update Data Makefile to build pipeline functions and resolvers

**Day 5: Distance/Time Goals** ✅ COMPLETE (2026-03-07)
- ✅ Database fields already exist: `target_value`, `target_unit`, `target_frequency`
- ✅ Distance/time goals use same mechanism as frequency goals (accumulate value vs target per period)
- ✅ Activity logging UI supports value + unit input
- ✅ Progress calculated as accumulated value vs target
- ✅ Example goals seeded: "10,000 Steps a Day", "Cycle to Work 3x per Week"


### Week 4: Super Goals & Wainwright Regions

**Day 1-2: Super Goals Structure**
- ⬜ Add `parent_goal_id` column to goals table
- ⬜ Create `super_goal` goal_type
- ⬜ Auto-calculate super goal progress from child goals
- ⬜ Auto-complete super goal when all children complete

**Day 3-4: Seed Super Goals**
- ⬜ Create "Visit All NT Sites (UK)" super goal linking 11 regional goals
- ⬜ Split Wainwrights into 7 regional goals:
  - Northern Fells (24), North Western (29), Western (33)
  - Central (27), Southern (30), Eastern (35), Far Eastern (36)
- ⬜ Create "Complete All Wainwright Regions" super goal

**Day 5: Testing & Polish**
- ⬜ Test all goal types work correctly
- ⬜ Fix any bugs
- ⬜ Deploy to dev

### Week 5: Badges & Rewards

**Day 1-2: Badge System**
- ⬜ Create `user_badges` table migration (if not exists)
- ⬜ Define badge trigger system (on goal completion, activity count, etc.)
- ⬜ Auto-award badges on triggers
- ⬜ Seed initial badges:
  - ⬜ "Founding Member" - joined during beta stage
  - ⬜ "First Steps" - joined first public goal
  - ⬜ "Getting Started" - logged first activity
  - ⬜ "Completionist" - completed first goal
  - ⬜ "Explorer" - completed first location-based goal
  - ⬜ "Consistent" - maintained weekly goal for 4+ weeks
  - ⬜ "Dedicated" - maintained daily goal for 7+ days
  - ⬜ "Super Achiever" - completed a super goal
  - ⬜ "Wainwright Explorer" - completed any Wainwright peak
  - ⬜ "Wainwright Master" - completed all 214 Wainwrights
  - ⬜ "NT Explorer" - completed any NT region
  - ⬜ "NT Completionist" - completed all UK NT sites

**Day 3-4: Badge Display**
- ⬜ Show earned badges on profile
- ⬜ Badge notification when earned
- ⬜ Celebrate with confetti animation!
- ⬜ Award "Founding Member" badge to all existing users

**Day 5: Review & Plan Next Phase**
- ⬜ Review progress with Lynsey
- ⬜ User testing feedback
- ⬜ Plan social features (following, reactions)

### Week 6: Social Features (Moved from Week 3)

**Day 1-2: Activity Feed & Following**
- ⬜ Create `user_follows` table migration
- ⬜ Show activities from followed users
- ⬜ Show activities from goals you've joined
- ⬜ Follow/unfollow functionality

**Day 3-4: Reactions & Encouragement**
- ⬜ Add reactions/cheers to activities
- ⬜ Notification when someone cheers you
- ⬜ Public profile page with goals and progress

**Day 5: Polish**
- ⬜ Mobile responsive testing
- ⬜ Performance optimization
- ⬜ Deploy to dev

---

## 🔧 Technical Debt / Refactoring

### AppSync JS Runtime Limitations

**📖 See full documentation:** [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md)

Quick reference - when writing AppSync resolvers, be aware of these limitations:

| ❌ Not Supported | ✅ Use Instead |
|------------------|----------------|
| `for` / `for...of` / `for...in` loops | `forEach()`, `map()`, `filter()`, `reduce()` |
| `++` / `--` operators | `+= 1` / `-= 1` |
| Regex literals `/pattern/` | String methods or `new RegExp()` |
| `async` / `await` | Resolver model handles async |
| `try` / `catch` | Check `ctx.error` in response |

**Key rules:**
- **Max 2 SQL statements** in `createMySQLStatement()` — use pipeline resolvers for more
- **MySQL DATETIME format:** `YYYY-MM-DD HH:MM:SS` (not ISO 8601)
- **Pipeline resolvers:** Use `ctx.stash` to pass data between functions

### Explore Page Pagination
- ⬜ Implement proper server-side pagination for public goals
  - Currently fetching first 100 goals with client-side filtering
  - As goal count grows, need proper pagination with limit/offset
  - Add "Load More" or infinite scroll functionality
  - Move search/filter to server-side for better performance
- ⬜ Add pagination to other list views (Goals, Activity Feed, etc.)

### Resolver Organization ✅ COMPLETE (2026-03-03)
- [x] Reorganize resolvers by model/domain
  - `resolvers/src/users/` - getUser, listUsers, createUser, updateUser, deleteUser
  - `resolvers/src/goals/` - getGoal, listAllGoals, listPublicGoals, createGoal, updateGoal, deleteGoal, getUserGoal, listMyGoals, joinGoal, leaveGoal, listGoalItems
  - `resolvers/src/activities/` - logGoalActivity, listUserGoalActivities, getActivityFeed, deleteActivity, listAllActivities, logActivity
  - `resolvers/src/badges/` - getBadge, listAvailableBadges, listMyBadges, listBadges
  - `resolvers/src/categories/` - listCategories, getCategory
  - `resolvers/src/goal-types/` - listGoalTypes, getGoalType
  - `resolvers/src/events/` - event resolvers
  - `resolvers/src/orders/` - order resolvers
  - `resolvers/src/organisers/` - organiser resolvers
  - `resolvers/src/reactions/` - reaction resolvers
- [x] Update Makefile/build process to handle nested resolver structure
- [x] Update appsync-rds.ts to scan nested directories

---

## 🔜 Priority Tasks

### Database Schema: Collection Goals

We need to support "collection goals" where users tick off items (e.g., Wainwrights, NT locations).

**Proposed Schema Changes:**

1. **`goal_items`** (new table) - Items to tick off within a goal
   - `id`, `goal_id`, `name`, `description`
   - `location_lat`, `location_lng` (optional, for maps)
   - `external_id` (for linking to external sources)
   - `metadata` (JSON for flexible data: elevation, opening hours, etc.)
   - `display_order`, `created_at`

2. **`user_goals`** (rename `goal_participants`) - User's participation in a goal
   - `id`, `user_id`, `goal_id`
   - `joined_at`, `completed_at`, `status`

3. **`user_goal_activities`** (new table) - Visits/completions of goal items
   - `id`, `user_goal_id`, `goal_item_id`
   - `activity_date`, `is_completion` (boolean - ticks off the item)
   - `notes`, `photos` (JSON array of URLs)
   - `location_lat`, `location_lng` (where they actually were)
   - `created_at`

**How It Works:**
- Goal "Summit all Wainwrights" has 214 `goal_items`
- User joins goal → creates `user_goals` entry
- User logs visit → creates `user_goal_activities` with `is_completion = true`
- Progress = `COUNT(DISTINCT goal_item_id WHERE is_completion = true)` / total items
- Same item can be visited multiple times (multiple Helvellyn summits!)

**First Goals to Create:**
- ⬜ Summit all Wainwrights (214 peaks)
- ⬜ Visit all National Trust locations (North East region)

**Questions to Resolve:**
- ⬜ Keep existing `activities` table for social feed separate from `user_goal_activities`?
- ⬜ How to seed the 214 Wainwrights data? (CSV import, API, manual?)
- ⬜ How to seed NT locations? (NT API, scrape, manual?)

---

### Data Quality & Curation

- ⬜ **National Trust Data Curation (Pre-Production)**
  - Current data is from OSM (buildings/infrastructure, not official visitor properties)
  - Manually curate official NT visitor properties list
  - Sources to consider:
    - National Trust official website property list
    - NT membership handbook
    - Manual data entry of ~300-400 main properties
  - Replace current 548 OSM-sourced locations
  - Verify locations have correct lat/lng coordinates
  - Ensure descriptions match visitor expectations

### Web App Improvements

- ⬜ **Create New Goal page** - form to create personal goals
- ⬜ **Goal Detail page** - view goal, items, progress, log activities
- ⬜ **Explore page** - browse and join public goals
- ⬜ **Settings page** - profile settings, privacy, notifications
- ⬜ **Add form validation** using Zod schemas from shared package
- ⬜ **Improve error handling** - show user-friendly messages
- ⬜ **Add loading skeletons** instead of spinners for better UX
- ⬜ **Mobile responsive testing** - ensure all pages work on mobile

### Admin Console

- ⬜ **CRUD for Goals** - create/edit/delete goals with items
- ⬜ **CRUD for Goal Items** - manage items within goals
- ⬜ **CRUD for Badges** - create/edit badges with criteria
- ⬜ **User management** - view users, assign admin role
- ⬜ **Seed data import** - CSV upload for Wainwrights/NT locations
- ⬜ **Analytics dashboard** - active users, goals joined, completions

### Documentation

- ⬜ **API documentation** - document all GraphQL queries/mutations
- ⬜ **README updates** - setup instructions for each repo
- ⬜ **Architecture diagram** - visual overview of system
- ⬜ **Onboarding guide** - for new contributors (future)

---

### Infrastructure & DevOps

- ⬜ **Set up AWS Budget Alerts**
  - Create budget in AWS Console: `https://console.aws.amazon.com/billing/home#/budgets`
  - Set threshold (e.g., $20/month) with email notifications
  - Can also add via Pulumi in `goals-club-data/packages/infra`

- ⬜ **Initial Git commits for all repos**
  - `goals-club-data` - .gitignore committed, need to commit remaining files
  - `goals-club-shared` - .gitignore committed
  - `goals-club-admin` - .gitignore committed
  - `goals-club-web` - .gitignore committed

- ⬜ **Set up GitHub/GitLab remotes**
  - Create remote repositories
  - Add origins: `git remote add origin <url>`
  - Push initial commits

---

## 📝 Notes

### Cost Management Strategy
- Use `pulumi down` to destroy stacks when not actively developing
- Stacks up: ~$0.30-$5/day (depending on DB usage)
- Stacks down: ~$0/month (only CodeArtifact storage ~$0.50/month)

### Aurora Serverless v2 Configuration
- Min capacity: 0 ACU (scales to zero)
- Auto-pause: 300 seconds (5 minutes)
- When paused: $0 compute cost

---

## ✅ Completed

- [x] Initialize git repos for Data, Shared, Admin, Web (2026-03-01)
- [x] Rename branches from `master` to `main` (2026-03-01)
- [x] Remove redundant `.env.frontend` generation from Data infra (2026-03-01)
- [x] AWS cost analysis completed (2026-03-02)
- [x] Replace all demo data with real GraphQL API calls (2026-03-02)
  - Goals page → `listMyGoals`
  - ActivityFeed → `getActivityFeed`
  - Badges → `listAvailableBadges` + `listMyBadges`
  - Profile → `getMe`
- [x] Fix datetime format for MySQL in mutation resolvers (2026-03-02)
- [x] Create public badge queries for web users (2026-03-02)
- [x] Migrate DNS from Cloudflare to Route 53 (2026-03-02)
  - Route 53 hosted zone created: `Z09867643V33MOCM1W0RD`
  - Pulumi now manages DNS records alongside CloudFront
  - SES DKIM records auto-created in Route 53
- [x] Admin user creation as part of data deployment (2026-03-02)
  - Admin users created automatically during `make deploy`
  - Users added to Cognito "Admins" group
  - Temporary password: `TempPass123!` (change on first login)
- [x] Resolver reorganization by model/domain (2026-03-03)
- [x] First activity logged - Catbells summit (2026-03-03)
- [x] Seed National Trust locations across 11 UK regions (2026-03-04)
- [x] Add Mapbox GL map view for location-based goals (2026-03-04)
- [x] Multiple visits to same item support (2026-03-07)
- [x] Conditional Map tab display for location-based goals only (2026-03-07)
- [x] Frequency goals with streak tracking (2026-03-07)
  - Pipeline resolver implementation (insertActivity → getGoalPeriodInfo → upsertGoalPeriod)
  - Period History UI with visual week boxes showing date (d/m) + ✓/✗
  - Streak statistics display (current streak, longest streak, periods completed)
  - Fixed MySQL datetime formatting issues (use `YYYY-MM-DD HH:MM:SS` not ISO format)
- [x] Updated Data Makefile to build pipeline functions and resolvers (2026-03-07)

---

## 🏔️ Activity Data to Seed on Stack Rebuild

When you rebuild the stack (`make destroy` / `make deploy`), remember to log these activities:

| Goal Item | Date | Notes |
|-----------|------|-------|
| Catbells | 12/04/2023 | Really popular, wore brand new Van boots, bad mistake. Enjoyed the views and the dachshunds, did not enjoy the foot pain. |

*Add more completions to this list as you log them!*

---

## 🔧 Route 53 DNS Migration

**Action Required:** Update nameservers at your domain registrar (OVH) to point to Route 53.

**Route 53 Nameservers:**
```
ns-1439.awsdns-51.org
ns-1546.awsdns-01.co.uk
ns-260.awsdns-32.com
ns-910.awsdns-49.net
```

**Steps:**
1. Log into OVH domain management
2. Find DNS/Nameserver settings for thegoalsclub.co.uk
3. Replace existing nameservers with the Route 53 ones above
4. Wait for propagation (up to 48 hours, usually faster)
5. Verify with: `dig thegoalsclub.co.uk NS`

**Benefits:**
- `make destroy` / `make deploy` now works without manual DNS updates
- CloudFront + DNS records are created/destroyed together
- SES DKIM records are auto-managed
