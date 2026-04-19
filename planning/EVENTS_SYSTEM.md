# Events System

## Overview

The Goals Club supports **events** - organised activities that users can express interest in or log participation. Events are managed by **organisers** which may be claimed or unclaimed.

## Key Concepts

### Organisers

Organisers are entities that run events (companies, charities, or individuals).

**Claimed vs Unclaimed:**
- **Unclaimed**: We create the organiser profile with their public info. No `user_id` is linked. The organiser can later "claim" their account to manage their events.
- **Claimed**: An organiser with a linked user account who can manage their events on the platform.

**Organiser Fields:**
| Field | Description |
|-------|-------------|
| `user_id` | Nullable - linked user account (null if unclaimed) |
| `organisation_name` | Display name |
| `slug` | URL-friendly identifier |
| `description` | About the organiser |
| `website_url` | Official website |
| `logo_url` | Logo image |
| `contact_email` | For reaching out (private) |
| `status` | pending, approved, rejected, suspended, trusted |
| `is_claimed` | Boolean flag |
| `can_self_publish` | If true, events auto-approve |

### Event Series

A **series** groups related events together. Examples:
- "parkrun" series contains all weekly parkrun events
- "Abbott World Marathon Majors" contains all six major marathons
- "Great North Run" contains annual editions (2024, 2025, 2026...)

**Series Fields:**
| Field | Description |
|-------|-------------|
| `organiser_id` | Parent organiser |
| `name` | Series name |
| `slug` | URL-friendly identifier |
| `description` | About the series |
| `category_id` | Activity category |
| `is_recurring` | True for weekly/annual events |
| `recurrence_pattern` | "weekly", "annual", etc. |

### Events

Individual event instances (a specific date/location).

**Event Fields:**
| Field | Description |
|-------|-------------|
| `organiser_id` | Parent organiser |
| `series_id` | Optional parent series |
| `name` | Event name |
| `slug` | URL-friendly identifier |
| `description` | Event details |
| `category_id` | Activity category |
| `event_date` | Start date |
| `event_end_date` | End date (for multi-day events) |
| `location_lat/lng` | Coordinates |
| `location_name` | Human-readable location |
| `external_url` | Official event page |
| `distance_km` | Distance (for running/cycling) |
| `event_type` | 5k, 10k, half_marathon, marathon, etc. |
| `status` | draft, pending, approved, rejected, archived |

## Seeded Organisers

| Organiser | Type | Events |
|-----------|------|--------|
| **parkrun** | Weekly 5K runs | 2000+ locations worldwide |
| **The Great Run Company** | Mass participation | Great North Run, Great South Run, etc. |
| **SuperHalfs** | European half marathons | Lisbon, Prague, Copenhagen, Cardiff, Valencia |
| **Abbott World Marathon Majors** | Elite marathons | Tokyo, Boston, London, Berlin, Chicago, NYC |
| **T100 Triathlon World Tour** | Pro triathlon | Global tour events |
| **London Marathon Events** | UK running | TCS London Marathon, The Big Half |
| **IRONMAN** | Triathlon | IRONMAN, IRONMAN 70.3 series |

## User Interactions

### Expressing Interest (Shortlist)
Users can shortlist events they're considering:
- "Shortlist" → adds to `user_event_interests` with status `INTERESTED`
- Shown as "Shortlisted" badge on the event

### "I'm Doing This" (Commit)
When a user commits to an event:
- Calls `joinGoal` on the event's canonical goal (`goals.event_id = event.id`)
- Creates a `user_goals` record linking user to the canonical goal
- Status is checked by querying `listMyGoals` for a matching `goalId` with `status = ACTIVE`
- Shows a "View Goal" card on the event detail page
- Shows a "View Event" card on the goal detail page (bidirectional navigation)

### Canonical Event Goals
When an admin approves an event:
- The `createCanonicalGoalForEvent` pipeline function auto-creates a `goals` record
- Goal type: `event`, visibility: `PUBLIC`, title = event name
- `events.goal_id` is updated to link back to the goal
- Users who commit join this shared canonical goal

### Rejoining
If a user leaves (abandons) an event goal:
- The `joinGoal` mutation uses `ON DUPLICATE KEY UPDATE status = 'ACTIVE'`
- Users can rejoin from the Abandoned tab in My Goals or from the event detail page

## Claiming an Organiser Account

When an organiser wants to claim their account:

1. They register on the platform
2. Submit a claim request with verification
3. Admin reviews and approves
4. `user_id` is linked, `is_claimed` set to true
5. Organiser can now manage their events

## Event Types

| Type | Distance | Description |
|------|----------|-------------|
| `5k` | 5 km | Fun runs, parkrun |
| `10k` | 10 km | Popular road race distance |
| `half_marathon` | 21.1 km | Half marathon |
| `marathon` | 42.195 km | Full marathon |
| `ultra` | > 42.2 km | Ultramarathon |
| `triathlon_sprint` | ~25 km | Sprint triathlon |
| `triathlon_olympic` | ~51.5 km | Olympic distance triathlon |
| `half_ironman` | 113 km | IRONMAN 70.3 |
| `ironman` | 226 km | Full IRONMAN |

## Future Enhancements

- [ ] parkrun location database (2000+ locations)
- [ ] Automatic event date updates from external sources
- [ ] Event reviews and ratings
- [ ] Training plans linked to events
- [ ] Event reminders and notifications
- [ ] Social features (who's doing this event?)
- [ ] Virtual events support

