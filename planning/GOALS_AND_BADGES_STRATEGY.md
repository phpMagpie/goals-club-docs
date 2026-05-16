# Goals & Badges Strategy

> Created: 9 May 2026
> Status: **Seeders + pipeline rewritten ✅ — badge images pending**

---

## Overview

The new badge design reference (`BADGE_DESIGN_REFERENCE.md`) introduced a completely new badge system — different animals, names, levels, and criteria from what was originally seeded. This document maps the gaps and defines what needs to change in the database and goals content to support the new system.

---

## Badge System Summary (new design)

12 categories, 5 levels each (~60 total) + 6 meta/prestige badges:

| Category | Animal | Colour | Maps to Goal Category |
|---|---|---|---|
| Goals Joined | 🐦 Robin | Warm Red `#E4572E` | (cross-cutting) |
| Activity Logged | 🐿️ Red Squirrel | Rust Orange `#D97A2B` | (cross-cutting) |
| Streaks | 🐢 Tortoise | Moss Green `#5C8A3D` | (cross-cutting) |
| Support Given | 🐕 Golden Retriever | Golden Yellow `#E3B23C` | (cross-cutting, reactions) |
| Goal Completion | 🐝 Bee | Honey Yellow `#F2C94C` | (cross-cutting) |
| Collections | 🐦 Magpie | Charcoal `#444444` | Outdoor Challenges + Collection Goals |
| Run | 🐇 Hare | Light Brown `#C08457` | Running |
| Walk | 🐕 Spaniel | Soft Blue `#6FA8DC` | Walking |
| Hike | 🐑 Sheep | Stone Grey `#9E9E9E` | **Hiking (new category)** |
| Swim | 🦦 Otter | Teal `#2A9D8F` | Swimming |
| Cycle | 🦊 Fox | Burnt Orange `#C75B12` | Cycling |
| Meta/Prestige | 🐐 Goat | Deep Indigo `#2F2A5A` | (combination-based) |

---

## Gap Analysis

### 1. Badge seeder is completely wrong ❌

The current seeder has entirely different badges — old names, old animals, wrong criteria. **Full rewrite needed.**

| Current (old seeder) | New design |
|---|---|
| First Steps (1 goal joined) | First Perch (1 goal joined) — Robin |
| High Five (5 goals joined) | Taking Flight (3 goals joined) — Robin |
| Ambitious (10 goals joined) | Finding Your Path (7 goals) — Robin |
| Getting Started (1 activity) | First Acorn (1 activity) — Squirrel |
| Active (25 activities) | Getting Busy (10 activities) — Squirrel |
| etc. | etc. |

### 2. "Walking & Hiking" category mixes two badge types ❌

Currently one category (`walking-hiking`) covers both casual walks (→ Spaniel) and hikes/peaks (→ Sheep). These are different badge tracks and need to be tracked separately.

**Fix:** Split into two categories:
- `walking` — daily/casual walks, dog walks, step goals → Spaniel badges
- `hiking` — Wainwrights, hill walks, long-distance routes → Sheep badges

### 3. No dedicated hiking goals seeded ❌

All hiking content is tied to the Wainwrights collection (a mega-goal). No standalone recurring hiking goals exist to help users earn Sheep badges.

### 4. Activity type tracking (future) ⚠️

Currently there is no `activity_type` field on activities beyond the broad `ActivityType` enum (`ITEM_COMPLETION`, `PROGRESS_LOG`, `VISIT`). The badge criteria for Run/Walk/Hike/Swim/Cycle need to be awarded based on which *goal category* the activity was logged against. The `checkAndAwardBadges` resolver will need to join `user_goal_activities → user_goals → goals → categories` to count per-discipline activities.

---

## Categories — Required Changes

### Split "Walking & Hiking" into two

| New Category | Slug | Icon | Badge track |
|---|---|---|---|
| Walking | `walking` | 🚶 | Spaniel |
| Hiking | `hiking` | 🥾 | Sheep |

Existing recurring goals under `walking-hiking`:
- Walk 10,000 Steps Daily → `walking` ✅
- Walk the Dog Twice Daily → `walking` ✅
- Walk Kids to School 5x per Week → `walking` ✅
- 5 Long Dog Walks per Week → `walking` ✅

Wainwrights goal → `hiking` ✅
(NT goal stays under `outdoor-challenges` → Magpie/collection track)

### New hiking goals to add

| Goal | Type | Target | Frequency |
|---|---|---|---|
| Complete a Hike Every Week | Recurring | 1 | weekly |
| Walk a New Trail Every Month | Recurring | 1 | monthly |
| Complete a Half Day Hike Every Month | Recurring | 1 | monthly |
| Hike 50km in a Month | Recurring | 50 km | monthly |

---

## Goals Content Assessment

### ✅ Recurring goals — good set, minor category fix needed

| Goal | Current Category | Correct Category |
|---|---|---|
| Run 5K Every Week | running-events | running-events ✅ |
| Run 10K Every Week | running-events | running-events ✅ |
| Run 25K Every Week | running-events | running-events ✅ |
| Walk 10,000 Steps Daily | walking-hiking | walking (new) |
| Walk the Dog Twice Daily | walking-hiking | walking (new) |
| Walk Kids to School 5x/week | walking-hiking | walking (new) |
| 5 Long Dog Walks per Week | walking-hiking | walking (new) |
| Cycle 100K Every Month | cycling ✅ | cycling ✅ |
| Cycle to Work 3x per Week | cycling ✅ | cycling ✅ |
| Swim 1K Every Week | swimming ✅ | swimming ✅ |
| Swim 3x per Week | swimming ✅ | swimming ✅ |
| Meditate 10 Minutes Daily | fitness-milestones | habit-goals |
| Exercise 30 Minutes Daily | fitness-milestones | fitness-milestones ✅ |
| Workout 4x per Week | fitness-milestones | fitness-milestones ✅ |
| Read for 30 Minutes Daily | habit-goals ✅ | habit-goals ✅ |
| Drink 8 Glasses of Water Daily | habit-goals ✅ | habit-goals ✅ |

### ✅ Collection goals — solid

- Wainwrights (214 peaks) — very popular, move to `hiking`
- National Trust (by region) — stays under `outdoor-challenges`

### ⬜ Missing — Milestone goals

No milestone goals currently seeded. Good candidates for onboarding modal "popular goals":

| Goal | Type | Description |
|---|---|---|
| Complete a 5K Race | Milestone | First parkrun or 5K event |
| Run a Half Marathon | Milestone | 21.1km training target |
| Run a Marathon | Milestone | 42.2km — the big one |
| Cycle 100 Miles (Century Ride) | Milestone | Classic cycling challenge |
| Complete a Triathlon | Milestone | Swim/bike/run combined |
| Learn to Swim 1 Mile | Milestone | Open water or pool |
| Walk the Coast to Coast | Milestone | Alfred Wainwright's 192-mile route |
| Complete the Yorkshire Three Peaks | Milestone | 24-mile classic challenge |
| Climb Ben Nevis | Milestone | UK's highest peak |
| Complete a Tough Mudder | Milestone | Obstacle course challenge |

---

## Badge Seeder Rewrite Plan

### Level thresholds (new design)

**Goals Joined (Robin)**
- L1 First Perch — 1 goal
- L2 Taking Flight — 3 goals
- L3 Finding Your Path — 7 goals
- L4 Path Explorer — 15 goals
- L5 Journey Maker — 30 goals

**Activity Logged (Squirrel)**
- L1 First Acorn — 1 activity
- L2 Getting Busy — 10 activities
- L3 In Motion — 25 activities
- L4 Always Moving — 75 activities
- L5 Relentless Energy — 150 activities

**Streaks (Tortoise)**
- L1 Getting Going — 3-day streak
- L2 Steady Steps — 7-day streak
- L3 In Rhythm — 14-day streak
- L4 Unshakeable — 30-day streak
- L5 Legendary Pace — 90-day streak

**Support Given (Golden Retriever)**
- L1 Kind Gesture — 1 reaction
- L2 Friendly Face — 5 reactions
- L3 Reliable Friend — 15 reactions
- L4 Community Builder — 40 reactions
- L5 Heart of the Club — 100 reactions

**Goal Completion (Bee)**
- L1 First Honey — 1 completion
- L2 Sweet Success — 3 completions
- L3 Hive Builder — 10 completions
- L4 Master Worker — 25 completions
- L5 Hive Leader — 50 completions

**Collections (Magpie)**
- L1 First Find — 1 collection
- L2 Curious Collector — 3 collections
- L3 Treasure Seeker — 7 collections
- L4 Keen Gatherer — 15 collections
- L5 Master Collector — 30 collections

**Run (Hare)**
- L1 First Dash — 1 run
- L2 Finding Pace — 5 runs
- L3 Quick Stride — 15 runs
- L4 Fleet Footed — 40 runs
- L5 Wind Chaser — 100 runs

**Walk (Spaniel)**
- L1 First Walk — 1 walk
- L2 Out and About — 5 walks
- L3 Daily Rambler — 15 walks
- L4 Trail Regular — 40 walks
- L5 Endless Wanderer — 100 walks

**Hike (Sheep)**
- L1 First Hill — 1 hike
- L2 Hill Walker — 5 hikes
- L3 Ridge Roamer — 12 hikes
- L4 Mountain Regular — 30 hikes
- L5 High Grounder — 75 hikes

**Swim (Otter)**
- L1 First Dip — 1 swim
- L2 Making Waves — 5 swims
- L3 In the Flow — 15 swims
- L4 Strong Current — 40 swims
- L5 Water Master — 100 swims

**Cycle (Fox)**
- L1 First Ride — 1 ride
- L2 Rolling Along — 5 rides
- L3 Smooth Operator — 15 rides
- L4 Road Runner — 40 rides
- L5 Trail Strategist — 100 rides

**Meta/Prestige (Goat)**
- Sure Footed — 5 goals completed
- Peak Performer — 25 activities + 5 completions
- Mountain Mindset — 14-day streak + 10 activities
- All-Terrain — activities in 3+ disciplines
- Summit Seeker — 10 goals completed
- The Goat — 50 goals completed

---

## Image file naming (new system)

All badge images follow: `/badges/{category}-{level}.png`

Examples:
- `/badges/robin-l1.png` → First Perch
- `/badges/robin-l2.png` → Taking Flight
- `/badges/squirrel-l1.png` → First Acorn
- `/badges/founding-member.png` → Special (already exists ✅)

---

## Action Plan

| # | Task | Where | Priority | Status |
|---|---|---|---|---|
| 1 | Rewrite badge seeder with new 67-badge system | `goals-club-data` | 🔴 Now | ✅ Done |
| 2 | Split `walking-hiking` → `walking` + `hiking` categories | `goals-club-data` | 🔴 Now | ✅ Done |
| 3 | Add hiking recurring goals (4 new) | `goals-club-data` | 🟡 Soon | ✅ Done |
| 4 | Add milestone goals seeder (14 goals) | `goals-club-data` | 🟡 Soon | ✅ Done |
| 5 | Update `getUserBadgeStats` — add discipline counts, reactions, collections, All-Terrain | `goals-club-data` | 🔴 Now | ✅ Done |
| 6 | Update `awardQualifyingBadges` checkCriteria — all new types + multi_criteria | `goals-club-data` | 🔴 Now | ✅ Done |
| 7 | Update Wainwrights seeder — category `walking-hiking` → `hiking` | `goals-club-data` | 🔴 Now | ✅ Done |
| 8 | Generate badge placeholder images (67 files) | `goals-club-web` | 🟡 Soon | ⬜ Pending |
| 9 | Update badge UI to show level + category grouping | `goals-club-web` | 🟡 Soon | ⬜ Pending |
| 10 | Generate real badge art using DALL·E prompts from NEW_BADGE_DESIGN_REFERENCE | External | 🟢 Post-launch | ⬜ Pending |

