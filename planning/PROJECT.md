# The Goals Club - Project Overview

## Vision

A social platform where people can set, share, and encourage others to complete meaningful personal goals. Building a supportive community focused on physical and mental health improvement through the creation of local "Goal Clubs".

---

## Core User Types

### 1. Goal Setters (Regular Users)
- Register and create a minimal profile
- Create personal goals (milestone, recurring, or collection-based)
- Join other people's shared goals ("I'd like to do that too!")
- Log activity/progress with photos
- Share progress and receive encouragement
- Earn virtual and physical badges/patches

### 2. Event Organisers
- Register (requires admin approval)
- Create and manage events (initial approval required, trusted organisers can self-publish later)
- Allow users to join events
- Revenue opportunity for platform sustainability

### 3. Local Goal Club Leaders (future)
- Facilitate local goal-achievement communities
- Organise meetups and group activities
- Built organically from geographic proximity + shared goal interests

---

## Goal Types

| Type | Description | Examples |
|------|-------------|----------|
| **Milestone** | One-time completion | "Complete Great North Run", "First 5K" |
| **Recurring** | Repeated target | "Walk 5km every week", "10,000 steps daily" |
| **Collection** | Tick off items from a list | Wainwrights (214 peaks), National Trust sites |
| **Event** | Linked to organised event | London Marathon, Parkrun |

---

## Goal Categories

1. Running Events
2. Walking/Hiking
3. Cycling
4. Swimming
5. Fitness Milestones
6. Outdoor Challenges
7. Collection Goals
8. Habit Goals
9. Other

---

## Core Features

### User Profiles
| Field | Required | Displayed |
|-------|----------|-----------|
| Username | ✅ Yes | ✅ Public |
| Display Name | ❌ Optional | ✅ If set |
| Profile Photo | ❌ Optional | ✅ If set |
| Location | ✅ Yes (for Goal Clubs) | ⚙️ User choice |
| Bio | ❌ Optional | ✅ If set |

### Goal Management
- Goal creation with title, description, category, type
- Privacy settings: Private / Shared / Public
- Target tracking (date-based or frequency-based)
- Progress history and visualisation
- Archive/delete functionality

### Progress Logging
- Activity date and notes
- Photo uploads (optional)
- Value tracking for recurring goals
- Completion marking for milestones
- Location tagging (manual or from photo EXIF)

### Social Features (Privacy-First)
- All content **private by default**
- User-level and goal-level privacy controls
- Following disabled by default (opt-in)
- Pre-determined reaction set for encouragement
- Activity feed from followed users/goals

### Badges & Achievements
- Virtual badges auto-awarded on completion
- Badge gallery on profile
- Physical merchandise available for completed goals (post-MVP)

---

## Reaction Types

| Reaction | Emoji | Use Case |
|----------|-------|----------|
| Cheer | 👏 | General encouragement |
| High Five | 🙌 | Celebrating progress |
| Impressed | 🤩 | Wow moments |
| Keep Going | 💪 | Motivation boost |
| Love It | ❤️ | Appreciation |
| Inspired | ✨ | When someone inspires you |

*Users can suggest additional reactions — moderated before adoption*

---

## Revenue Model

### MVP: Free Platform
- Full platform access
- Virtual badges/patches
- Unlimited goals
- **Focus:** Build community first, monetize later

### Post-MVP Revenue Streams (Phase 2)
1. **Physical Merchandise** - Pins, badges, patches for goal completion
   - *Requires:* Merch partner setup (print-on-demand or supplier)
2. **Event Organiser Fees** - Subscription or per-event fees
3. **Badge Marketplace** (future) - Commission on partner badge sales
4. **Premium Features** (potential) - Advanced analytics, custom badges

---

## Success Metrics

### Launch Targets
- 100 registered users in first month
- 50 goals created in first month
- 10 goals with multiple participants
- 20 virtual badges earned

### Engagement Metrics
- Daily active users
- Goals created per user
- Progress logs per goal
- Reactions given per user
- Return visit rate

---

## Post-MVP Features

These features are planned but deferred until after launch:

| Feature | Reason for Deferral |
|---------|---------------------|
| Physical Merchandise Shop | Need merch partners first, build community |
| Event Organisers | Focus on user goals first |
| Strava integration | Requires live site for OAuth review |
| Apple Health / Fitbit / Garmin | Phase 2 integration |
| Local Goal Clubs | Needs organic community first |
| Accountability Partners | Post-launch feature |
| Goal Templates | Can add after seeing real user goals |
| Time-Limited Challenges | Post-launch engagement feature |
| Badge Marketplace | Needs community scale |
| Comments/Messaging | Minimal until community trust established |
| Advanced visualisations | Basic progress view first |
| Mobile apps | Web-first, responsive design |

---

## Technical Stack

See [`TECHNICAL_ARCHITECTURE.md`](./TECHNICAL_ARCHITECTURE.md) for full details.

| Service | Purpose |
|---------|---------|
| Aurora Serverless v2 | MySQL database (scales to zero) |
| AppSync | GraphQL API with real-time subscriptions |
| Cognito | User authentication |
| S3 + CloudFront | Static hosting, CDN |
| SES | Transactional email |

---

*Document consolidated: March 8, 2026*
*Original sources: PROJECT_OVERVIEW.md, MVP_SCOPE.md*

