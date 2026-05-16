# The Goals Club — Database Schema

> Auto-generated from squashed migration `20260419000001-squashed-initial-schema.js`
> Last updated: 2026-05-11

## Overview

Complete MySQL database schema for The Goals Club platform (26 tables). Supports:
- **Template goals** (public/joinable) and **custom goals** (personal)
- **Collection goals** with tickable items (Wainwrights, National Trust sites)
- **Recurring goals** with period tracking and streaks
- **Event goals** linked to organised events and event series
- **Strava integration** with OAuth tokens, activity dedup, and goal linking
- **Social features** — follows, reactions, activity feed
- **Badges** with auto-awarding criteria
- **E-commerce** for physical badge merchandise

## Visual ERD

Paste [`schema.dbml`](./schema.dbml) into [dbdiagram.io](https://dbdiagram.io) for an interactive diagram.

---

## Lookup Tables

### goal_types

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(50) | NOT NULL |
| slug | VARCHAR(50) | UNIQUE |
| description | VARCHAR(200) | |
| icon | VARCHAR(50) | |
| display_order | INT | Default 0 |

**Seeds:** milestone, recurring, collection, event

### categories

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(100) | NOT NULL |
| slug | VARCHAR(100) | UNIQUE |
| description | TEXT | |
| icon | VARCHAR(50) | Emoji |
| color | VARCHAR(7) | Hex |
| display_order | INT | Default 0 |
| active | BOOLEAN | Default true |

### reaction_types

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(50) | NOT NULL |
| emoji | VARCHAR(10) | NOT NULL |
| active | BOOLEAN | Default true |
| display_order | INT | Default 0 |

---

## Core Tables

### users

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| cognito_id | VARCHAR(128) | UNIQUE |
| email | VARCHAR(255) | UNIQUE |
| username | VARCHAR(50) | UNIQUE, nullable |
| display_name | VARCHAR(100) | |
| bio | TEXT | |
| avatar_url | VARCHAR(500) | |
| location_lat | DECIMAL(10,8) | |
| location_lng | DECIMAL(11,8) | |
| location_name | VARCHAR(200) | |
| show_location | BOOLEAN | Default false |
| allow_followers | BOOLEAN | Default true |
| follower_visibility | ENUM('friends','anyone') | Default 'friends' |
| default_visibility | ENUM('PRIVATE','FRIENDS','PUBLIC') | Default 'PRIVATE' |
| suspended | BOOLEAN | Default false, NOT NULL |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### goals

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| creator_id | UUID | FK → users, nullable, CASCADE |
| event_id | UUID | FK → events, nullable |
| title | VARCHAR(200) | NOT NULL |
| description | TEXT | |
| category_id | UUID | FK → categories, SET NULL |
| goal_type_id | UUID | FK → goal_types, NOT NULL, RESTRICT |
| target_value | DECIMAL(10,2) | For recurring goals |
| target_unit | VARCHAR(50) | km, times, hours, etc. |
| target_frequency | ENUM('daily','weekly','monthly','yearly','once') | |
| target_date | DATE | For milestone/event goals |
| visibility | ENUM('PUBLIC','PRIVATE','FRIENDS') | Default 'PUBLIC' |
| allow_joins | BOOLEAN | Default true |
| total_items | INT | Default 0, cached count |
| image_url | VARCHAR(500) | |
| status | ENUM('ACTIVE','COMPLETED','ARCHIVED','DRAFT') | Default 'ACTIVE' |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Indexes:** creator_id, event_id, visibility, status, category_id, goal_type_id, (allow_joins + visibility + status)

### goal_items

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| goal_id | UUID | FK → goals, NOT NULL, CASCADE |
| name | VARCHAR(200) | NOT NULL |
| description | TEXT | |
| location_lat | DECIMAL(10,8) | For map view |
| location_lng | DECIMAL(11,8) | |
| location_name | VARCHAR(200) | |
| external_id | VARCHAR(100) | e.g. Wainwright # |
| metadata | JSON | Elevation, region, etc. |
| image_url | VARCHAR(500) | |
| display_order | INT | Default 0 |

### user_goals

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| goal_id | UUID | FK → goals, CASCADE, nullable |
| custom_title | VARCHAR(200) | For custom/personal goals |
| custom_description | TEXT | |
| custom_category_id | UUID | FK → categories |
| custom_goal_type_id | UUID | FK → goal_types |
| custom_target_value | DECIMAL(10,2) | |
| custom_target_unit | VARCHAR(50) | |
| custom_target_frequency | ENUM('daily','weekly','monthly','yearly','once') | |
| custom_target_date | DATE | |
| visibility | ENUM('PUBLIC','PRIVATE','FRIENDS') | Default 'PRIVATE' |
| status | ENUM('ACTIVE','COMPLETED','PAUSED','ABANDONED') | Default 'ACTIVE' |
| current_progress | DECIMAL(10,2) | Default 0 |
| items_completed | INT | Default 0 |
| current_streak | INT | Default 0, NOT NULL |
| longest_streak | INT | Default 0, NOT NULL |
| last_period_completed_at | DATETIME | nullable |
| periods_completed | INT | Default 0, NOT NULL |
| joined_at | DATETIME | |
| completed_at | DATETIME | nullable |
| updated_at | DATETIME | |

**Unique constraint:** (user_id, goal_id)

**Design:** `goal_id IS NOT NULL` = joined template goal; `goal_id IS NULL` = custom/personal goal

### user_goal_activities

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_goal_id | UUID | FK → user_goals, NOT NULL, CASCADE |
| activity_type | ENUM('ITEM_COMPLETION','PROGRESS_LOG','VISIT') | NOT NULL |
| goal_item_id | UUID | FK → goal_items, SET NULL |
| strava_activity_id | UUID | FK → strava_activities, SET NULL |
| value | DECIMAL(10,2) | Progress amount |
| unit | VARCHAR(50) | |
| activity_date | DATE | NOT NULL |
| is_completion | BOOLEAN | Default false |
| notes | TEXT | |
| location_lat | DECIMAL(10,8) | |
| location_lng | DECIMAL(11,8) | |
| location_name | VARCHAR(200) | |
| photos | JSON | Array of URLs |
| created_at | DATETIME | |

### user_goal_periods
Tracks recurring goal progress per time period. Used for streak calculation.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_goal_id | UUID | FK → user_goals, NOT NULL, CASCADE |
| period_start | DATE | NOT NULL |
| period_end | DATE | NOT NULL |
| period_type | ENUM('DAILY','WEEKLY','MONTHLY') | NOT NULL |
| target_value | DECIMAL(10,2) | NOT NULL |
| achieved_value | DECIMAL(10,2) | Default 0, NOT NULL |
| target_met | BOOLEAN | Default false, NOT NULL |
| consecutive_periods_met | INT | Default 0, NOT NULL |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Unique constraint:** (user_goal_id, period_start, period_type)

---

## Social Tables

### user_follows

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| follower_id | UUID | FK → users, NOT NULL, CASCADE |
| following_id | UUID | FK → users, NOT NULL, CASCADE |
| created_at | DATETIME | |

**Unique constraint:** (follower_id, following_id)

### goal_follows

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| goal_id | UUID | FK → goals, NOT NULL, CASCADE |
| created_at | DATETIME | |

**Unique constraint:** (user_id, goal_id)

### reactions
One reaction per user per activity. Changing emoji replaces the previous reaction.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| activity_id | UUID | FK → user_goal_activities, NOT NULL, CASCADE |
| reaction_type_id | UUID | FK → reaction_types, NOT NULL, CASCADE |
| created_at | DATETIME | |

**Unique constraint:** (user_id, activity_id)

---

## Badges & Achievements

### badges

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(100) | NOT NULL |
| description | TEXT | |
| image_url | VARCHAR(500) | |
| badge_type | ENUM('system','goal','event','category','special','prestige') | NOT NULL |
| category_id | UUID | FK → categories, SET NULL |
| goal_id | UUID | FK → goals, SET NULL |
| criteria | JSON | Auto-award criteria definition |
| is_limited_edition | BOOLEAN | Default false |
| active | BOOLEAN | Default true |

### user_badges

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| badge_id | UUID | FK → badges, NOT NULL, CASCADE |
| user_goal_id | UUID | FK → user_goals, SET NULL |
| earned_at | DATETIME | |

**Unique constraint:** (user_id, badge_id)

---

## Events & Organisers

### organisers

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, nullable, SET NULL |
| name | VARCHAR(200) | NOT NULL |
| slug | VARCHAR(100) | UNIQUE, nullable |
| description | TEXT | |
| website_url | VARCHAR(500) | |
| logo_url | VARCHAR(500) | |
| contact_email | VARCHAR(255) | nullable |
| status | ENUM('pending','approved','rejected','suspended','trusted') | Default 'pending' |
| is_claimed | BOOLEAN | Default false, NOT NULL |
| can_self_publish | BOOLEAN | Default false |
| approved_at | DATETIME | |
| approved_by | UUID | |
| rejection_reason | TEXT | |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### event_series
Groups related events (e.g. "Great North Run" series under Great Run Company).

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| organiser_id | UUID | FK → organisers, NOT NULL, CASCADE |
| name | VARCHAR(200) | NOT NULL |
| slug | VARCHAR(100) | UNIQUE, NOT NULL |
| description | TEXT | |
| category_id | UUID | FK → categories, SET NULL |
| image_url | VARCHAR(500) | |
| website_url | VARCHAR(500) | |
| external_url | VARCHAR(500) | |
| is_recurring | BOOLEAN | Default false |
| recurrence_pattern | VARCHAR(50) | e.g. "annual" |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### events

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| organiser_id | UUID | FK → organisers, NOT NULL, CASCADE |
| goal_id | UUID | FK → goals, nullable, SET NULL |
| series_id | UUID | FK → event_series, nullable, SET NULL |
| name | VARCHAR(200) | NOT NULL |
| slug | VARCHAR(200) | nullable |
| description | TEXT | |
| category_id | UUID | FK → categories, SET NULL |
| event_date | DATE | |
| event_end_date | DATE | |
| location_lat | DECIMAL(10,8) | |
| location_lng | DECIMAL(11,8) | |
| location_name | VARCHAR(200) | |
| location_address | VARCHAR(500) | |
| distance_km | DECIMAL(10,2) | nullable |
| event_type | VARCHAR(50) | nullable |
| external_url | VARCHAR(500) | |
| website_url | VARCHAR(500) | |
| registration_url | VARCHAR(500) | |
| registration_close_date | DATE | |
| max_participants | INT | nullable |
| price_pence | INT | nullable |
| image_url | VARCHAR(500) | |
| status | ENUM('draft','pending','approved','rejected','archived') | Default 'pending' |
| rejection_reason | TEXT | |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Note:** When an event is approved, a canonical goal is auto-created and linked via `goal_id`. Users "join" the event by joining this goal.

### user_event_interests

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| event_id | UUID | FK → events, NOT NULL, CASCADE |
| status | ENUM('interested','committed') | Default 'interested' |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Unique constraint:** (user_id, event_id)

---

## Strava Integration

### strava_tokens
OAuth tokens for connected Strava accounts. One per user.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, UNIQUE, NOT NULL, CASCADE |
| strava_athlete_id | BIGINT | NOT NULL |
| access_token | VARCHAR(255) | NOT NULL |
| refresh_token | VARCHAR(255) | NOT NULL |
| expires_at | INT | Unix timestamp, NOT NULL |
| scope | VARCHAR(255) | |
| created_at | DATETIME | |
| updated_at | DATETIME | |

**Indexes:** user_id (unique), strava_athlete_id, expires_at

### strava_goal_links
Maps Strava activity types to user goals for auto-sync.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| user_goal_id | UUID | FK → user_goals, NOT NULL, CASCADE |
| strava_activity_type | VARCHAR(50) | e.g. "Run", "Ride", NOT NULL |
| created_at | DATETIME | |

**Unique constraint:** (user_goal_id, strava_activity_type)

### strava_activities
Dedup table — prevents processing the same Strava activity twice.

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| strava_activity_id | BIGINT | UNIQUE, NOT NULL |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| strava_activity_type | VARCHAR(50) | |
| distance_meters | FLOAT | |
| moving_time_seconds | INT | |
| start_date | DATETIME | |
| name | VARCHAR(255) | |
| start_lat | DECIMAL(10,8) | GPS latitude of activity start |
| start_lng | DECIMAL(11,8) | GPS longitude of activity start |
| route_polyline | TEXT | Google Encoded Polyline from Strava |
| processed_at | DATETIME | |
| created_at | DATETIME | |

---

## E-Commerce Tables

### products

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| name | VARCHAR(200) | NOT NULL |
| description | TEXT | |
| price_pence | INT | NOT NULL |
| image_url | VARCHAR(500) | |
| badge_id | UUID | FK → badges, SET NULL |
| product_type | ENUM('badge','patch','pin','other') | Default 'badge' |
| stock_quantity | INT | Default 0 |
| stock_status | ENUM('in_stock','low_stock','out_of_stock','pre_order') | Default 'in_stock' |
| is_limited_edition | BOOLEAN | Default false |
| active | BOOLEAN | Default true |
| stripe_product_id | VARCHAR(100) | |
| stripe_price_id | VARCHAR(100) | |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### orders

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| status | ENUM('pending','paid','processing','shipped','delivered','cancelled','refunded') | Default 'pending' |
| total_pence | INT | NOT NULL |
| stripe_payment_intent_id | VARCHAR(100) | |
| shipping_name | VARCHAR(200) | |
| shipping_address_line1 | VARCHAR(200) | |
| shipping_address_line2 | VARCHAR(200) | |
| shipping_city | VARCHAR(100) | |
| shipping_postcode | VARCHAR(20) | |
| shipping_country | VARCHAR(2) | |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### order_items

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| order_id | UUID | FK → orders, NOT NULL, CASCADE |
| product_id | UUID | FK → products, NOT NULL, RESTRICT |
| quantity | INT | Default 1, NOT NULL |
| unit_price_pence | INT | NOT NULL |
| created_at | DATETIME | |

---

## System Tables

### notifications

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, NOT NULL, CASCADE |
| type | VARCHAR(50) | NOT NULL |
| title | VARCHAR(200) | |
| message | TEXT | |
| data | JSON | |
| read_at | DATETIME | NULL = unread |
| created_at | DATETIME | |

### audit_log

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| admin_user_id | UUID | |
| action | VARCHAR(100) | NOT NULL |
| resource_type | VARCHAR(50) | |
| resource_id | UUID | |
| old_values | JSON | |
| new_values | JSON | |
| ip_address | VARCHAR(45) | |
| created_at | DATETIME | |
