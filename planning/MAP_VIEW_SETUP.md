# Map View Setup

## Mapbox Token

The map view uses Mapbox GL JS for rendering interactive maps of goal locations (Wainwrights, National Trust sites, etc.).

### Getting a Free Mapbox Token

1. **Sign up** for a free Mapbox account: https://account.mapbox.com/auth/signup/
2. **Create an access token**:
   - Go to https://account.mapbox.com/access-tokens/
   - Click "Create a token"
   - Name it "Goals Club Dev" or similar
   - Copy the token (starts with `pk.`)
3. **Add to environment**:
   - Add `VITE_MAPBOX_TOKEN=your_token_here` to `goals-club-web/packages/ui/.env`
   - Or set as environment variable when running dev server

### Free Tier Limits

Mapbox free tier includes:
- **50,000 map views per month** (plenty for development)
- Unlimited tokens
- All map styles and features

### Fallback Token

The app uses a Mapbox demo token by default (`mapbox` account public token), which works for development but has rate limits. Replace with your own token for production.

## Features

The map view displays:
- ✅ **Green markers** for completed items
- 🔵 **Blue markers** for pending items
- **Popups** with item names and descriptions
- **Auto-fit bounds** to show all markers
- **Progress stats** overlay showing completion count
- **Legend** explaining marker colors

## Implementation

**Component:** `goals-club-web/packages/ui/src/components/GoalItemsMap.tsx`

**Usage:**
```tsx
<GoalItemsMap 
  items={goalItems}              // Array of GoalItem with locationLat/Lng
  completedActivities={activities} // Array of UserGoalActivity
  height="600px"                 // Optional, defaults to 500px
/>
```

**Map Style:** `outdoors-v12` - Shows terrain, trails, and elevation (perfect for hiking/outdoor goals)

## Development

The map component:
- Only renders if items have valid `locationLat` and `locationLng` coordinates
- Calculates bounds automatically to fit all markers
- Filters activities to only show completions
- Uses React hooks for lifecycle management
- Cleans up map instance on unmount

## Alternative Map Providers

If you prefer a different provider:
- **Leaflet** (free, no token required, but less features)
- **Google Maps** (requires API key, paid after quota)
- **OpenStreetMap** (completely free, but DIY setup)

Mapbox was chosen for:
- Excellent outdoor/terrain styles
- Good performance with 500+ markers
- Beautiful default styling
- Active development and support

