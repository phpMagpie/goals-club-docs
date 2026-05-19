# Group Challenges — Feature Spec

> Status: **Phase 1 Complete** (May 17, 2026) — live on dev. Designed as a B2B2C revenue feature.

---

## Overview

Enable businesses (e.g. sports massage therapists, gyms, running clubs) to create branded community challenges with invite links, shared goals, and leaderboards.

**Use Case:** A sports massage therapist creates a "Run 100km This Month" challenge, shares an invite link with clients, and tracks their progress on a leaderboard. Prizes for hitting milestones (e.g. "Free massage for anyone who completes 100km").

---

## Data Model

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

---

## Join Flow

1. Group admin creates group → gets invite link: `thegoalsclub.co.uk/groups/join/MASSAGE2026`
2. User clicks link → must be logged in → sees group description + goals → "Join Group"
3. User joins group as member — does **not** auto-join group goals
4. User can explicitly join individual group goals via "Join" button on goal cards
5. User appears on leaderboard for joined goals

---

## Leaderboard

- No new table — aggregation query on `user_goal_activities` for group members
- `getGroupLeaderboard(groupId, goalId, sortBy, limit, startDate, endDate)`
- Monthly filtering via `startDate`/`endDate` params — UI provides month picker (last 12 months)
- Falls back to `group_goals.start_date/end_date` when no explicit dates, then all-time
- Returns: `[{ user, totalValue, currentStreak, longestStreak, rank }]`
- Sortable by: total progress, current streak, longest streak

---

## Architecture Notes

- Reuses existing goals, activities, streaks, badges — groups are a social wrapper
- Group activity feed = existing feed filtered by group members
- Invite code flow requires authentication (moved from public to authenticated layout)
- Group admin is just a user with elevated role in that context
- Privacy: members see each other's progress on group goals only — doesn't expose other goals
- Any user can create a group (no restrictions on group creation)

---

## Key Decisions

- **No built-in chat** — link to WhatsApp/Discord via `chat_url` field
- **No auto-join** — members explicitly opt into individual goals via "Join" button (admins may not participate; members may only want some goals)
- **Time-bounded challenges** — `start_date`/`end_date` on `group_goals` scopes the leaderboard, plus query-time `startDate`/`endDate` for monthly views
- **Multiple admins** — `role: ADMIN` in `group_members`
- **Abuse prevention** — rate-limit group creation, admin can suspend groups

---

## Phasing

| Phase | Scope | Status |
|-------|-------|--------|
| **Phase 1** | Groups + members + invite link + group goals + leaderboard (monthly filtering) + group feed + join notifications | ✅ Complete |
| **Phase 2** | Milestones/prizes + badge awards for group achievements | Planned |
| **Phase 3** | Paid groups (Stripe) + group admin analytics dashboard | Planned |
| **Phase 4** | Public group directory + featured groups | Planned |

---

## Revenue Model

- **Free tier:** 1 group, up to 20 members, 3 goals
- **Pro tier (£9.99/mo):** Unlimited groups, custom branding, milestone prizes, analytics
- **Enterprise:** White-label, API access, SSO

