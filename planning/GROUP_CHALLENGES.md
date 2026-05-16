# Group Challenges — Feature Spec

> Status: **Future** — not yet started. Designed as a B2B2C revenue feature.

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
2. User clicks link → sees group description + goals → "Join Group"
3. Joining auto-joins all group goals (or pick from list)
4. User appears on leaderboard immediately

---

## Leaderboard

- No new table — aggregation query on `user_goal_activities` for group members
- `getGroupLeaderboard(groupId, goalId, period: "all" | "month" | "week")`
- Returns: `[{ user, totalValue, currentStreak, rank }]`
- Sortable by: total progress, current streak, longest streak

---

## Architecture Notes

- Reuses existing goals, activities, streaks, badges — groups are a social wrapper
- Group activity feed = existing feed filtered by group members
- Invite code flow similar to event "I'm Doing This"
- Group admin is just a user with elevated role in that context
- Privacy: members see each other's progress on group goals only — doesn't expose other goals

---

## Key Decisions

- **No built-in chat** — link to WhatsApp/Discord via `chat_url` field
- **Time-bounded challenges** — `start_date`/`end_date` on `group_goals` scopes the leaderboard
- **Multiple admins** — `role: ADMIN` in `group_members`
- **Abuse prevention** — rate-limit group creation, admin can suspend groups

---

## Phasing

| Phase | Scope |
|-------|-------|
| **Phase 1** | Groups + members + invite link + group goals + basic leaderboard |
| **Phase 2** | Milestones/prizes + badge awards for group achievements |
| **Phase 3** | Paid groups (Stripe) + group admin analytics dashboard |
| **Phase 4** | Public group directory + featured groups |

---

## Revenue Model

- **Free tier:** 1 group, up to 20 members, 3 goals
- **Pro tier (£9.99/mo):** Unlimited groups, custom branding, milestone prizes, analytics
- **Enterprise:** White-label, API access, SSO

