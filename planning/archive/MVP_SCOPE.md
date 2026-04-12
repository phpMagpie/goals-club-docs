# The Goals Club - MVP Scope

## Overview
This document defines the Minimum Viable Product (MVP) for The Goals Club - the smallest feature set needed to launch, gather feedback, and prove the concept.

---

## MVP Guiding Principles

1. **Trust-based first** - No external integrations required at launch
2. **Simple over complex** - Basic features done well
3. **Community foundation** - Enable the social aspects that make this unique
4. **Revenue-ready** - Physical badge shop functional from day one
5. **Privacy-conscious** - Users control their visibility

---

## MVP Features

### ✅ User Registration & Profiles

**Registration**
- Email/password registration
- Email verification
- Terms & privacy policy acceptance

**Profile (Minimal)**
- Username (required, unique)
- Display name (optional)
- Profile photo (optional)
- Location (required for system, display optional)
- Short bio (optional)
- Privacy setting: Show location on profile (yes/no)

**Profile View**
- Public profile page
- List of public goals
- Badges earned
- Member since date

---

### ✅ Goals

**Goal Creation**
- Goal title
- Description
- Category selection (from predefined list)
- Goal type:
  - **Milestone** - One-time completion (e.g., "Complete Great North Run")
  - **Recurring** - Repeated target (e.g., "Walk 5km every week")
- Target (flexible based on type):
  - Milestone: Completion date or open-ended
  - Recurring: Frequency (daily/weekly/monthly) + target
- Privacy setting:
  - **Private** - Only visible to creator
  - **Shared** - Visible to selected users
  - **Public** - Visible to all + joinable by others

**Goal Management**
- Edit goal details
- Archive/delete goal
- Mark as complete
- View progress history

**Joining Goals**
- Browse public goals
- "Join this goal" button
- See others who've joined the same goal
- Leave a joined goal

---

### ✅ Progress Logging

**Activity Logging**
- Date of activity
- Description/notes
- Photo upload (optional, up to 3 photos)
- For recurring goals: Progress value (e.g., "5km walked")
- For milestone goals: Progress update or completion

**Progress Feed**
- Personal activity history
- Photo gallery from activities

---

### ✅ Social Features (Minimal)

**Privacy-First Defaults**
- All goals and activities are **private by default**
- Users can opt-in to sharing at account or goal level
- Following is **disabled by default**

**Sharing Settings (User Level)**
- Default: Private (no sharing)
- Option: Share with friends only
- Option: Share with anyone

**Sharing Settings (Goal Level)**
- Inherits user default, OR
- Override per-goal (private/friends/public)

**Following (Opt-in)**
- Following disabled by default
- User must enable "Allow followers" in settings
- Options when enabled:
  - Friends only can follow
  - Anyone can follow
- Same options available per-goal

**Encouragement System**
- Pre-determined selection of reactions (user chooses which to use)
- Users can suggest new reactions (moderated before adoption)
- See reaction counts on activities
- Notification when receiving encouragement

**Activity Feed**
- Chronological feed of shared activities from followed users/goals
- Only shows content user has permission to see

---

### ✅ Badges & Achievements

**Virtual Badges**
- Auto-awarded on goal completion
- Badge gallery on profile
- Badge detail page (what it's for, when earned)

**Initial Badge Set**
- "First Goal Created"
- "First Goal Completed"
- Category-specific completion badges
- "Community Member" (joined first shared goal)
- "Encourager" (gave first cheer)
- "Founder" (early adopter badge)

---

### ✅ Physical Merchandise Shop

**Shop Basics**
- View available badges/patches for completed goals
- Product pages with images and pricing
- Add to cart
- Basic checkout flow
- Order confirmation

**Order Management**
- Order history
- Order status tracking

**Payment Processing**
- Stripe integration for checkout
- Support for cards and common payment methods

---

### ✅ Admin Panel

**User Management**
- View all users
- Suspend/ban users
- View user details

**Event Organiser Management**
- Review organiser applications
- Approve/reject organisers
- View organiser list

**Event Management**
- Review submitted events
- Approve/reject events
- Edit event details

**Basic Moderation**
- Report review queue
- Content moderation tools

---

### ✅ Event Organisers (Basic)

**Organiser Registration**
- Application form
- Admin approval required

**Event Creation** (approved organisers only)
- Event name
- Description
- Date(s)
- Location
- Category
- External link (to official event page)
- Submit for approval

**Event Listing**
- Public event directory
- Search/filter events
- Event detail pages

---

## MVP Categories (Predefined)

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

## Explicitly NOT in MVP

These features are planned but will come after initial launch:

| Feature | Reason for Deferral |
|---------|---------------------|
| Strava integration | Requires live site for OAuth review |
| Apple Health / Fitbit / Garmin | Phase 2 integration |
| Local Goal Clubs | Needs organic community first |
| Accountability Partners | Post-launch feature |
| Goal Templates | Can add after seeing real user goals |
| Time-Limited Challenges | Post-launch engagement feature |
| Badge Marketplace | Needs community scale |
| Comments/Messaging | Minimal interaction until community trust established |
| Advanced visualisations | Basic progress view first |
| Spectator/Supporter accounts | Can use regular accounts initially |
| Premium tier | Free to start, monetise via merchandise |
| Mobile apps | Web-first, responsive design |

---

## Success Metrics for MVP

### Launch Targets
- [ ] 100 registered users in first month
- [ ] 50 goals created in first month
- [ ] 10 goals with multiple participants
- [ ] 5 badge purchases

### User Engagement
- Daily active users
- Goals created per user
- Progress logs per goal
- Cheers given per user
- Return visit rate

### Revenue
- Badge/patch orders
- Average order value
- Organiser applications

---

## Technical Requirements for MVP

*To be detailed in TECHNICAL_ARCHITECTURE.md*

### Essentials
- Responsive web design (mobile-friendly)
- Fast page loads
- Secure authentication
- Image upload and storage
- Payment processing
- Email notifications
- Basic SEO

### Performance Targets
- Page load < 3 seconds
- Image upload < 10 seconds
- 99.9% uptime target

---

## MVP Timeline

*To be detailed in ROADMAP.md*

### Phases
1. **Foundation** - Auth, profiles, basic goals
2. **Social** - Following, cheers, activity feed
3. **Badges** - Virtual badges, shop integration
4. **Events** - Organiser flow, event directory
5. **Polish** - Testing, refinement, launch prep
6. **Launch** - Soft launch, gather feedback

---

*Document created: February 28, 2025*
*Last updated: February 28, 2025*

