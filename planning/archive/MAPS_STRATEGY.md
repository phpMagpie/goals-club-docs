# The Goals Club - Maps & Location Strategy

## Overview
This document outlines how The Goals Club can incorporate maps and location features **without requiring external fitness app integrations** in the MVP.

---

## The Challenge

Without Strava, Garmin, Apple Health integrations (which require a live site for OAuth approval), we need alternative approaches for:
1. Showing where activities took place
2. Visualising progress on location-based goals (e.g., Wainwrights, NT sites)
3. Building location data for future Goal Club features

---

## MVP Map Solutions

### Option 1: Manual Location Tagging (Recommended for MVP)

**How it works:**
- When logging activity, users can add a location
- Location picker using a map interface (Leaflet + OpenStreetMap - free)
- User taps/clicks to place a pin, or searches for a location
- Stores: lat/long, place name, optional notes

**Use cases:**
- "I walked up Helvellyn today" → Pin on Helvellyn summit
- "Completed Parkrun at Durham" → Pin on event location
- "Visited Cragside NT" → Pin on property

**Benefits:**
- No external integration needed
- Works offline (user adds location later)
- Simple to implement
- User-controlled data

**Limitations:**
- No automatic route tracking
- Relies on user effort
- Single point, not a route

---

### Option 2: Photo Location Data (EXIF)

**How it works:**
- When users upload photos with their activity log
- Extract GPS coordinates from photo EXIF data (if available)
- Auto-suggest location based on photo metadata
- User confirms or adjusts

**Benefits:**
- "Free" location data from photos users are already uploading
- Adds authenticity to check-ins
- Low user effort

**Limitations:**
- Only works if user has location enabled on camera
- Privacy considerations - must clearly explain we're extracting this
- Not all photos have location data

**Implementation:**
- PHP has built-in EXIF support
- Extract on upload, present to user for confirmation
- Never store without explicit consent

---

### Option 3: GPX File Upload (Power Users)

**How it works:**
- Users export GPX files from their fitness apps/devices
- Upload GPX when logging activity
- We parse and display the route on a map

**Benefits:**
- Full route data, not just a point
- Works with any device that exports GPX
- No OAuth integration needed
- Power users will appreciate this

**Limitations:**
- Extra steps for user
- Only tech-savvy users will use it
- Need to build GPX parser

**Implementation:**
- GPX is just XML - easy to parse
- Libraries available for PHP/JS
- Display with Leaflet polylines

---

## Recommended MVP Approach

### Phase 1 (MVP Launch)
1. **Manual location tagging** - Simple pin placement on map
2. **Photo EXIF extraction** - Auto-suggest from uploaded photos
3. **Location search** - Type to find places
4. **Screenshot sharing** - Encourage sharing of Strava/Garmin screenshots (see below)

### Phase 2 (Post-Launch)
4. **GPX file upload** - For power users wanting route data
5. **Strava integration** - Once site is live and approved

### Phase 3 (Future)
6. **Apple Health / Garmin / Fitbit** - Additional integrations based on user research
7. **Native app with GPS tracking** - Real-time route recording

---

## Screenshot Sharing Strategy

### Purpose
Encourage users to share screenshots from their existing fitness apps (Strava, Garmin Connect, Apple Fitness, etc.) when logging activities. This serves multiple purposes:

**User Benefits:**
- Share beautiful route maps and stats immediately
- Show verified data from trusted sources
- No extra effort - they're already taking these screenshots
- Visual proof of achievements for badges

**Platform Benefits:**
- Understand which services our users actually use
- Prioritise integration development based on real data
- Build case for integration partnerships
- Users get used to sharing activity data (smooth transition to auto-sync later)

### Implementation

**Activity Log Form:**
- Photo upload field accepts screenshots alongside regular photos
- Optional tag: "This is from: Strava / Garmin / Apple / Fitbit / Other"
- Helps us track which platforms to prioritise

**User Research Data:**
- Track which fitness apps are most commonly referenced
- Use this data to prioritise Phase 2/3 integrations
- Could even show users: "Join 47 others waiting for Garmin integration!"

**Community Aspect:**
- Screenshots showing pace, elevation, routes inspire others
- Creates aspirational content ("I want to do that route!")
- Builds social proof before we have native tracking

### Privacy Note
- Screenshots may contain personal data (name, location)
- Remind users to check what's visible before sharing
- Screenshots follow same privacy rules as other photos

---

## Map Technology Choices

### Display: Leaflet.js + OpenStreetMap
- **Cost:** Free and open source
- **Quality:** Excellent outdoor/hiking maps
- **Customisation:** Full control over styling
- **Offline:** Can cache tiles for specific areas

### Alternative: Mapbox
- **Cost:** Free tier (50,000 map loads/month), then paid
- **Quality:** Beautiful maps, great satellite imagery
- **Features:** More advanced styling, terrain data
- **Consideration:** Good for future if we scale

### NOT Recommended: Google Maps
- **Cost:** Expensive at scale
- **Quality:** Good but overkill for our needs
- **Terms:** Restrictive for our use case

---

## Location-Based Goal Visualisation

### Collection Goals (e.g., Wainwrights, NT Sites)

**Pre-loaded location data:**
- We can pre-load known locations for popular goals
- Wainwrights: 214 peaks with coordinates
- National Trust: ~500 properties with locations
- User checks off, we show on map

**Map Display:**
- Completed locations: Filled markers
- Remaining locations: Empty/grey markers
- Progress overlay: "47/214 Wainwrights"

### Distance Goals (e.g., Walk 100km this month)

**Without GPS tracking:**
- User logs distance manually
- Optional: Add location pin for where they walked
- Show all logged locations on a personal map

**Future with integrations:**
- Automatic distance from Strava/Garmin
- Route visualisation

---

## Privacy Considerations

### Location Data Handling
- Location always optional to add
- Clear explanation of how location is used
- Location display respects sharing settings:
  - Private goal = location never shown to others
  - Friends only = location visible to friends
  - Public = location visible on public profile/goal

### Photo EXIF Extraction
- Clearly notify users we can read photo location
- Explicit opt-in: "Use location from this photo?"
- Strip EXIF from stored photos (privacy)
- Only store location coordinates separately if user consents

---

## Data We'll Collect

| Data Point | Source | Required | Purpose |
|------------|--------|----------|---------|
| User home location | Registration | Yes (display optional) | Goal Clubs, local events |
| Activity location | Manual pin | Optional | Map visualisation |
| Photo location | EXIF extraction | Optional + confirmed | Auto-suggest location |
| GPX route | File upload | Optional | Route display |

---

## Technical Implementation Notes

### Libraries Needed
- **Leaflet.js** - Map display
- **OpenStreetMap tiles** - Base map layer
- **PHP EXIF functions** - Photo metadata extraction
- **GPX parser** - For route uploads (future)

### Database Storage
```
locations table:
- id
- locationable_type (polymorphic - Activity, Goal, etc.)
- locationable_id
- latitude
- longitude
- name (place name)
- created_at
```

### Performance Considerations
- Lazy load maps (don't load until visible)
- Cluster markers when zoomed out
- Cache tile requests where possible

---

## Summary

For MVP, we can deliver meaningful map functionality without any external integrations:

✅ **Manual location pins** - User places markers on activities
✅ **Photo location extraction** - Auto-suggest from uploaded images  
✅ **Pre-loaded goal locations** - Wainwrights, NT sites, etc. ready to go
✅ **Collection progress maps** - Visual "what I've done" displays
✅ **Leaflet + OSM** - Free, open source, great for outdoors
✅ **Screenshot sharing** - Encourage Strava/Garmin screenshots for visual proof + user research

This gives us a solid foundation that we enhance with Strava/Garmin integrations post-launch, prioritised based on what our users are actually using.

---

*Document created: February 28, 2025*

