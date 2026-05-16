# Pre-Launch Audit — Goals Club Web

> Created: 9 May 2026
> Updated: 10 May 2026
> Status: **All critical + important items resolved ✅ — badges partially done (l1/l2 live, l3–l5 pending)**

---

## 🔴 Critical (Must Fix Before Inviting Users)

### 1. Error Boundaries
- **Problem:** Any component crash shows a blank white screen. Users think the site is broken.
- **Fix:** Add a root `<ErrorBoundary>` with a friendly fallback UI and a "reload" button.
- **Status:** ✅ Done — `ErrorBoundary.tsx` created, wraps entire app in `App.tsx`

### 2. Loading / Empty States on Goals Page
- **Problem:** New users with no goals see a blank page — no guidance, no CTA.
- **Fix:** Add loading skeletons for query states, and an empty-state illustration/prompt ("Create your first goal") when the list is empty.
- **Status:** ✅ Already handled — Goals page has loading spinner, error alert, and "Create your first goal" empty state. Dashboard has skeletons, getting-started alert, and empty CTAs.

### 3. Mobile Navigation
- **Problem:** Header nav uses horizontal `Group` with no burger/drawer. Links overflow or get cut off on small screens.
- **Fix:** Add a Mantine `Burger` + `Drawer` for mobile breakpoints.
- **Status:** ✅ Done — `AppHeader.tsx` updated with `Burger` (hiddenFrom="sm") + `Drawer` with full nav, profile, settings, and sign-out

### 4. Sign-Out Confirmation / Feedback
- **Problem:** Users may accidentally tap sign out with no confirmation.
- **Fix:** Add a confirmation modal before signing out.
- **Status:** ✅ Done — Confirmation `Modal` added to `AppHeader.tsx` for both desktop menu and mobile drawer

---

## 🟡 Important (Should Fix Soon)

### 5. Badge Images — Placeholders
- **Problem:** Seeder references `.svg`/`.jpg` files that likely don't exist. Badge display shows broken images.
- **Fix:** Create or source actual badge artwork, or add a fallback/placeholder component.
- **Status:** ✅ Partially done — l1 + l2 images live for all 13 animal tracks (bee, border-terrier, fox, goat, hare, magpie, otter, retriever, robin, sheep, squirrel, tortoise) + founding-member. l3–l5 and goat-l6 fall back to placeholder.png (optimised). badge_type enum expanded to include `special` + `prestige` via migration. NT data replaced with fresh 618-place authoritative scrape across 10 regions.

### 6. User Onboarding Flow
- **Problem:** New user lands on dashboard with no goals, no prompts, nothing to do.
- **Fix:** Add a first-run onboarding stepper or welcome card that guides users to create a goal.
- **Status:** ✅ Done — `OnboardingModal.tsx` created with 2-step Mantine Stepper: (1) join a popular goal from a card grid or tap "Create my own", (2) connect Strava with skip option. Triggered from `AppLayout` after username setup + guarded by `localStorage` so it only shows once. Strava callback redirects to `/dashboard` post-connect.

### 7. GraphQL Error Handling
- **Problem:** `apolloClient.ts` has a basic error link but pages don't handle query errors gracefully.
- **Fix:** Add user-facing error states (toast notifications or inline error cards) for failed queries/mutations.
- **Status:** ✅ Done:
  - `graphql.ts` — improved to handle HTTP errors, network failures, and auth errors with typed exceptions (`GraphQLNetworkError`, `GraphQLAuthError`)
  - Error toasts added to all mutation hooks: `useCreateGoal`, `useUpdateUserGoal`, `useLeaveGoal`, `useJoinGoal`, `useLogGoalActivity`, `useStravaGoalLinks`
  - `CreateGoal.tsx` — **critical bug fixed**: form was not calling the API at all (`console.log` + navigate). Now fetches goal types, maps slugs to IDs, calls `useCreateGoal` + `useJoinGoal`, shows loading state, success/error toasts
  - `api.ts` — removed bare `console.*` calls

### 8. Tenant Data Isolation
- **Problem:** Must verify backend resolvers filter goals/activities by the authenticated user's ID.
- **Fix:** Audit AppSync resolvers to confirm `owner`/`userId` filtering is enforced.
- **Status:** ✅ Audited — two bugs found and fixed, overall isolation is solid:

**✅ Well-isolated resolvers (all enforce Cognito identity at DB level):**
- `listMyGoals`, `getUserGoal`, `updateUserGoal`, `leaveGoal`, `joinGoal` — all `WHERE ... u.cognito_id = cognitoId`
- `getActivityFeed` (MINE / FOLLOWING / ALL) — correct visibility + following filters
- `listUserGoalActivities`, `deleteGoalActivity`, `listUserGoalPeriods`, `updateGoalPeriodProgress` — ownership via JOIN
- All Strava resolvers — `WHERE u.cognito_id = cognitoId` throughout
- `listAllGoals`, `listAllActivities` — **Admin-only** at schema level (`@aws_cognito_user_pools(cognito_groups: ["Admins"])`)
- Default API auth mode is `AMAZON_COGNITO_USER_POOLS` — all mutations require a valid JWT

**🐛 Bugs fixed:**
1. **`getUserGoal.js`** — Missing explicit `util.error("Not authenticated", "Unauthorized")` guard. The WHERE clause still enforced isolation at DB level, but added the guard for defence in depth.
2. **`createGoal.js`** — Critical data integrity bug: was inserting `user_id = ctx.identity.sub` (Cognito sub UUID) into a `user_id` column. DB schema has `creator_id uuid [ref: > users.id]` — wrong column name AND wrong ID type. Fixed to `SELECT u.id FROM users WHERE u.cognito_id = ${cognitoId}` into `creator_id`, and `goal_type_id` (not the deprecated `goal_type` text column). Also fixed to use user's `default_visibility` as fallback.
3. **`followUser.js`** — Did not check `allow_followers` on the target user. Any user could be followed regardless of their privacy setting. Fixed: pre-query checks `allow_followers = 1`; if false returns a `Forbidden` error.
4. **`getActivityFeed.js`** — FOLLOWING and ALL filters did not check `allow_followers` on the users being followed. Added `AND u.allow_followers = 1` to all FOLLOWING-path JOINs (belt-and-suspenders; primary enforcement is now in `followUser`).
5. **`joinGoal.js`** — Ignored the user's `default_visibility` preference. Always stored `'PRIVATE'` if not explicitly set. Fixed: `COALESCE(NULLIF(input, ''), u.default_visibility, 'PRIVATE')` so the user's preference is respected.
6. **`OnboardingModal.tsx` + `useJoinGoal.ts`** — Frontend hardcoded `"PUBLIC"` (onboarding) and `"PRIVATE"` (hook default) overriding the user's preference. Changed to pass `""` so the backend COALESCE falls through to `default_visibility`.

---

## 🟢 Nice to Have (Post-Launch)

### 9. Reactions UI
- Backend exists, no frontend yet.
- **Status:** ✅ Done:
  - **Schema** — Added `emoji: String!` to `ReactionCount`, added `reactionCounts: [ReactionCount!]!` to `FeedItem`
  - **`getActivityFeed.js`** — Added correlated SQL subquery (`GROUP_CONCAT` across all 6 query variants) returning `reaction_summary` string parsed into `reactionCounts` in response
  - **`useActivities.ts`** — Added `reactionCounts` to GQL query and `FeedActivity` interface
  - **`useReactions.ts`** — New hook: module-level cached reaction types, per-activity `useReactions` with optimistic toggle (add/remove) + error revert
  - **`ReactionBar.tsx`** — New component: shows existing reactions as teal-highlighted emoji+count chips, smiley picker popover with all active types, scale animation on hover
  - **`ActivityFeed.tsx`** — `ReactionBar` wired into each activity card alongside the goal link

### 10. PWA / Mobile Install
- `webmanifest` exists but no service worker registered.
- **Status:** ✅ Done:
  - **`site.webmanifest`** — Fixed typo (`Glub` → `Club`), added `start_url`, `scope`, `orientation`, `description`, corrected `theme_color` to `#099268`, both `any` + `maskable` icon purposes
  - **`vite-plugin-pwa`** installed — generates `sw.js` + Workbox at build time; `registerType: "prompt"` so install is user-initiated, not automatic
  - **Workbox strategy** — no precaching (bundle too large; auth-gated so no offline value); runtime caching: `StaleWhileRevalidate` for JS/CSS chunks, `CacheFirst` for images, `NetworkOnly` for AppSync
  - **`InstallPrompt.tsx`** — fixed bottom banner: captures `beforeinstallprompt`, shows "Add to Home Screen" with Install / Not now buttons; dismissed state stored in localStorage; won't show again once dismissed
  - **Update nudge** — if a new SW version is detected, shows a "New version available — Refresh" banner
  - **`vite-env.d.ts`** created with `/// <reference types="vite-plugin-pwa/react" />` for virtual module types
  - SW auto-checks for updates every 60 minutes

### 11. SEO Meta Tags
- Only basic favicon tags — add Open Graph, description, etc.
- **Status:** ✅ Done:
  - Installed `react-helmet-async` — `HelmetProvider` wraps the app in `App.tsx`
  - `PageMeta` component (`src/components/PageMeta.tsx`) — accepts `title`, `description`, `ogImage`, `canonicalPath`, `noIndex`; renders full Open Graph + Twitter Card tags
  - `index.html` — upgraded with fallback OG/Twitter/theme-color tags for pre-JS rendering
  - All 15 pages tagged: Home (full OG + canonical), Privacy + Terms (canonical + noIndex), all auth-only pages (noIndex + descriptive titles)
  - Dynamic titles: GoalDetail uses goal title, EventDetail uses event name, Profile uses display name
  - `VITE_APP_URL` added to `.env.example` for canonical URL generation

---

## Progress Log

| # | Item | Priority | Date Completed | Notes |
|---|------|----------|----------------|-------|
| 1 | Error Boundaries | 🔴 Critical | 9 May 2026 | `ErrorBoundary.tsx` wraps full app |
| 2 | Loading / Empty States | 🔴 Critical | 9 May 2026 | ✅ Already existed on Goals + Dashboard |
| 3 | Mobile Navigation | 🔴 Critical | 9 May 2026 | Burger + Drawer added to `AppHeader` |
| 4 | Sign-Out Confirmation | 🔴 Critical | 9 May 2026 | Confirmation modal on all sign-out paths |
| 5 | Badge Images | 🟡 Important | 10 May 2026 | l1/l2 for all tracks live; l3–l5 use placeholder (acceptable) |
| 6 | Onboarding Flow | 🟡 Important | 9 May 2026 | `OnboardingModal` — join goal + Strava |
| 7 | GraphQL Error Handling | 🟡 Important | 9 May 2026 | Error toasts + `CreateGoal` API bug fixed |
| 8 | Tenant Data Isolation | 🟡 Important | 9 May 2026 | Audit passed; 6 bugs fixed across resolvers |
| 9 | Reactions UI | 🟢 Post-launch | 9 May 2026 | Full implementation — feed + hook + component |
| 10 | PWA / Mobile Install | 🟢 Post-launch | 9 May 2026 | vite-plugin-pwa + InstallPrompt banner |
| 11 | SEO Meta Tags | 🟢 Post-launch | 9 May 2026 | PageMeta + HelmetProvider — all 15 pages tagged |



