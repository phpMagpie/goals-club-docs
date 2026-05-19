# Group Challenges — Phase 1 Implementation Plan

Last updated: 2026-05-17

> **Status:** ✅ Phase 1 Complete — deployed to dev on May 17, 2026.

---

## Overview

Add groups, group members, group goals, invite-link join flow, leaderboard, group feed, and join notifications. Groups are a social wrapper around existing goals — no new goal system needed.

**Use Case:** Businesses (gyms, running clubs, massage therapists) create branded community challenges with invite links, shared goals, and leaderboards.

---

## Decisions

| Decision | Answer |
|----------|--------|
| Invite codes | Auto-generated (ULID substring), optional custom override. Must be unique (DB constraint) |
| Nav placement | Rethink nav — too many items already. Defer restructure or do as part of this work |
| Auto-join on addGroupGoal | **No** — removed during implementation. Members explicitly opt into goals via "Join" button |
| Auto-join on joinGroup | **No** — removed during implementation. Members choose which goals to join |
| Group activity feed | Yes — simple `getGroupActivityFeed` resolver in Phase 1 |
| Join notifications | Yes — `joinGroup` pipeline creates notification for group owner/admins |
| Branch strategy | Single branch for all Phase 1 work |
| Group creation restrictions | None — any authenticated user can create a group |
| Multiple goals per group | Yes — composite unique on (group_id, goal_id) |
| Monthly leaderboard | Query-time date filtering via `startDate`/`endDate` params (not per-month group_goals rows) |
| Invite join route | Authenticated layout (requires login before seeing group) |

---

## Database Migration

File: `packages/database/migrations/20260520000001-add-groups-tables.js`

### `groups`

| Column | Type | Constraints |
|--------|------|-------------|
| id | VARCHAR(26) | PK |
| name | VARCHAR(200) | NOT NULL |
| slug | VARCHAR(200) | UNIQUE, NOT NULL |
| description | TEXT | nullable |
| image_url | VARCHAR(500) | nullable |
| invite_code | VARCHAR(20) | UNIQUE, NOT NULL |
| creator_id | VARCHAR(26) | FK → users(id) SET NULL, nullable |
| chat_url | VARCHAR(500) | nullable |
| visibility | ENUM('PRIVATE','PUBLIC') | DEFAULT 'PRIVATE' |
| max_members | INT | nullable (null = unlimited) |
| start_date | DATE | nullable |
| end_date | DATE | nullable |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP |
| updated_at | DATETIME | DEFAULT CURRENT_TIMESTAMP ON UPDATE |

Indexes: `creator_id`, `invite_code` (unique), `slug` (unique), `visibility`

### `group_members`

| Column | Type | Constraints |
|--------|------|-------------|
| id | VARCHAR(26) | PK |
| group_id | VARCHAR(26) | FK → groups(id) CASCADE, NOT NULL |
| user_id | VARCHAR(26) | FK → users(id) CASCADE, NOT NULL |
| role | ENUM('OWNER','ADMIN','MEMBER') | DEFAULT 'MEMBER', NOT NULL |
| joined_at | DATETIME | DEFAULT CURRENT_TIMESTAMP |

Indexes: UNIQUE(group_id, user_id), `user_id`

### `group_goals`

| Column | Type | Constraints |
|--------|------|-------------|
| id | VARCHAR(26) | PK |
| group_id | VARCHAR(26) | FK → groups(id) CASCADE, NOT NULL |
| goal_id | VARCHAR(26) | FK → goals(id) CASCADE, NOT NULL |
| added_by | VARCHAR(26) | FK → users(id) SET NULL, nullable |
| leaderboard_enabled | BOOLEAN | DEFAULT true |
| start_date | DATE | nullable |
| end_date | DATE | nullable |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP |

Indexes: UNIQUE(group_id, goal_id), `goal_id`

---

## GraphQL Schema

### Enums

```graphql
enum GroupVisibility { PRIVATE PUBLIC }
enum GroupMemberRole { OWNER ADMIN MEMBER }
```

### Types

```graphql
type Group {
  id: ID!
  name: String!
  slug: String!
  description: String
  imageUrl: String
  inviteCode: String!
  creatorId: ID
  chatUrl: String
  visibility: GroupVisibility!
  maxMembers: Int
  startDate: AWSDate
  endDate: AWSDate
  createdAt: AWSDateTime!
  updatedAt: AWSDateTime!
  # Computed
  memberCount: Int!
  creator: User
}

type GroupMember {
  id: ID!
  groupId: ID!
  userId: ID!
  role: GroupMemberRole!
  joinedAt: AWSDateTime!
  user: User!
}

type GroupGoal {
  id: ID!
  groupId: ID!
  goalId: ID!
  addedBy: ID
  leaderboardEnabled: Boolean!
  startDate: AWSDate
  endDate: AWSDate
  createdAt: AWSDateTime!
  goal: Goal!
}

type GroupConnection {
  items: [Group!]!
  total: Int!
}

type GroupMemberConnection {
  items: [GroupMember!]!
  total: Int!
}

type LeaderboardEntry {
  userId: ID!
  displayName: String
  avatarUrl: String
  username: String
  totalValue: Float!
  currentStreak: Int!
  longestStreak: Int!
  rank: Int!
}

type GroupInviteInfo {
  group: Group!
  goals: [GroupGoal!]!
  memberCount: Int!
  isMember: Boolean
}
```

### Inputs

```graphql
input CreateGroupInput {
  name: String!
  description: String
  imageUrl: String
  chatUrl: String
  visibility: GroupVisibility
  maxMembers: Int
  startDate: AWSDate
  endDate: AWSDate
  inviteCode: String  # optional custom code
}

input UpdateGroupInput {
  name: String
  description: String
  imageUrl: String
  chatUrl: String
  visibility: GroupVisibility
  maxMembers: Int
  startDate: AWSDate
  endDate: AWSDate
}

input AddGroupGoalInput {
  groupId: ID!
  goalId: ID!
  leaderboardEnabled: Boolean
  startDate: AWSDate
  endDate: AWSDate
}
```

### Queries

```graphql
getGroup(id: ID!): Group @aws_cognito_user_pools
getGroupByInviteCode(inviteCode: String!): GroupInviteInfo @aws_api_key @aws_cognito_user_pools
listMyGroups(limit: Int, offset: Int): GroupConnection! @aws_cognito_user_pools
listGroupMembers(groupId: ID!, limit: Int, offset: Int): GroupMemberConnection! @aws_cognito_user_pools
listGroupGoals(groupId: ID!): [GroupGoal!]! @aws_cognito_user_pools
getGroupLeaderboard(groupId: ID!, goalId: ID!, sortBy: String, limit: Int, startDate: AWSDate, endDate: AWSDate): [LeaderboardEntry!]! @aws_cognito_user_pools
getGroupActivityFeed(groupId: ID!, limit: Int, nextToken: String): FeedConnection! @aws_cognito_user_pools
```

### Mutations

```graphql
createGroup(input: CreateGroupInput!): Group! @aws_cognito_user_pools
updateGroup(id: ID!, input: UpdateGroupInput!): Group! @aws_cognito_user_pools
deleteGroup(id: ID!): Boolean! @aws_cognito_user_pools
joinGroup(inviteCode: String!): GroupMember! @aws_cognito_user_pools
leaveGroup(groupId: ID!): Boolean! @aws_cognito_user_pools
addGroupGoal(input: AddGroupGoalInput!): GroupGoal! @aws_cognito_user_pools
removeGroupGoal(groupGoalId: ID!): Boolean! @aws_cognito_user_pools
updateMemberRole(groupId: ID!, userId: ID!, role: GroupMemberRole!): GroupMember! @aws_cognito_user_pools
```

---

## Resolvers

### Unit Resolvers (`resolvers/src/groups/`)

| Resolver | Type | Notes |
|----------|------|-------|
| getGroup | Query | SELECT + membership check for PRIVATE |
| getGroupByInviteCode | Query | Public/API key — SELECT group + goals + member count |
| listMyGroups | Query | JOIN group_members WHERE user, paginated |
| listGroupMembers | Query | Verify caller is member, paginated |
| listGroupGoals | Query | Verify caller is member |
| getGroupLeaderboard | Query | Aggregation on user_goal_activities (see below) |
| getGroupActivityFeed | Query | Existing feed SQL filtered by group members |
| updateGroup | Mutation | Verify OWNER/ADMIN |
| deleteGroup | Mutation | Verify OWNER only |
| leaveGroup | Mutation | Block if OWNER |
| removeGroupGoal | Mutation | Verify OWNER/ADMIN |
| updateMemberRole | Mutation | Verify OWNER only |

### Pipeline Resolvers (`pipeline-resolvers/src/`)

**`createGroup`** → `@functions=insertGroup,addOwnerMembership,fetchGroupResult`
- `insertGroup`: INSERT with auto ULID, generate invite_code, validate custom code uniqueness
- `addOwnerMembership`: INSERT group_members with role=OWNER
- `fetchGroupResult`: SELECT created group with member count

**`joinGroup`** → `@functions=validateJoinGroup,insertGroupMember,createGroupJoinNotification`
- `validateJoinGroup`: SELECT group by invite_code, COUNT members, check max, check not already member
- `insertGroupMember`: INSERT group_members role=MEMBER
- `createGroupJoinNotification`: INSERT notification for group owner
- *Note: `autoJoinGroupGoals` was removed — members explicitly opt into goals via "Join" button*

**`addGroupGoal`** → `@functions=insertGroupGoal`
- `insertGroupGoal`: INSERT group_goals, verify OWNER/ADMIN
- *Note: `autoJoinExistingMembers` was removed — members explicitly opt into goals via "Join" button*

### Leaderboard Query

Supports optional `startDate`/`endDate` for monthly filtering, falling back to `group_goals.start_date/end_date`:

```sql
SELECT
  u.id as user_id, u.display_name, u.avatar_url, u.username,
  COALESCE(SUM(uga.value), 0) as total_value,
  ug.current_streak, ug.longest_streak
FROM group_members gm
JOIN users u ON gm.user_id = u.id
JOIN group_goals gg ON gg.group_id = gm.group_id
JOIN user_goals ug ON ug.user_id = u.id AND ug.goal_id = gg.goal_id
LEFT JOIN user_goal_activities uga ON uga.user_goal_id = ug.id
  AND (:startDate IS NULL OR uga.activity_date >= COALESCE(:startDate, gg.start_date))
  AND (:endDate IS NULL OR uga.activity_date <= COALESCE(:endDate, gg.end_date))
WHERE gm.group_id = :groupId AND gg.goal_id = :goalId
GROUP BY u.id, u.display_name, u.avatar_url, u.username, ug.current_streak, ug.longest_streak
ORDER BY total_value DESC
LIMIT :limit
```

---

## Web App Pages

### Routes

| Route | Page | Auth |
|-------|------|------|
| `/groups` | Groups list | Auth'd |
| `/groups/new` | Create Group | Auth'd |
| `/groups/:id` | Group Detail (tabs: Goals, Leaderboard, Feed, Members) | Auth'd |
| `/groups/:id/settings` | Group Settings (edit, manage members, goals) | Auth'd (OWNER/ADMIN) |
| `/groups/join/:inviteCode` | Invite landing page | Auth'd (moved from public to authenticated layout) |

### Components

- `GroupCard.tsx` — Card for group listings
- `AddGoalToGroupModal.tsx` — Two-tab modal: "Search Existing" (debounced search of public goals, filters out already-linked) + "Create New" (inline goal creation with type/frequency/unit)
- `InviteLinkCopy.tsx` — Invite URL with copy button
- Leaderboard rendered inline in GroupDetail with month picker

### Hooks

- `useMyGroups` — list user's groups
- `useGroup` — single group detail (manages group, members, goals, leaderboard, feed, CRUD)
- `useMyGoals` — check which goals user has joined (for "Join"/"Joined" button state)
- `useJoinGoal` — join a goal from group context

### Invite Flow (authenticated)

1. User hits `/groups/join/MASSAGE2026`
2. Must be logged in (route is in authenticated layout — redirects to login if not)
3. Page calls `getGroupByInviteCode` — shows group info, goals, member count
4. "Join Group" button → calls `joinGroup` → redirect to `/groups/:id`
5. User lands on group detail — can then "Join" individual goals

---

## Edge Cases

| Case | Handling |
|------|----------|
| Max members reached | `validateJoinGroup` checks count vs max_members |
| Duplicate join | INSERT IGNORE (unique constraint), return existing membership |
| Owner leaves | Block with error: "Transfer ownership or delete the group" |
| Custom invite code collision | Validate uniqueness in `insertGroup`, error if taken |
| Private group access | All queries (except getGroupByInviteCode) verify membership |
| Expired group (past end_date) | Allow viewing/leaderboard, block new joins |
| Removing group goal | Deletes from group_goals only — user_goals preserved |
| Adding goal members already track | INSERT IGNORE INTO user_goals handles gracefully |

---

## Implementation Order

1. Database migration (3 tables)
2. GraphQL schema additions
3. Unit resolvers (12)
4. Pipeline: `createGroup`
5. Pipeline: `joinGroup` (with notification)
6. Pipeline: `addGroupGoal`
7. Leaderboard + group feed resolvers
8. Shared types (`@goals-club/shared`)
9. Web: Groups list + Create Group
10. Web: Group Detail (tabs: Goals, Leaderboard, Feed, Members)
11. Web: Group Settings
12. Web: Invite join flow (public route)
13. Nav restructure

