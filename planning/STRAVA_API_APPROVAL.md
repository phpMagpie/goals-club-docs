# Strava API Approval

## Current Status

The Strava integration is live and working for the app owner's account. The Strava API app is currently in **development mode**, which limits connections to **1 athlete** (not the documented 15). To allow unlimited users to connect their Strava accounts, the app must be submitted for Strava API approval.

**Approval submitted:** 2026-04-21 — Email sent to Strava with screenshots and app details.

---

## What Strava Requires for Approval

| Requirement | Status |
|---|---|
| App icon (square image) | ✅ Uploaded (Goals Club logo) |
| App description — what it does, how it uses Strava data | ✅ Included in email |
| Website URL — live site (dev.thegoalsclub.co.uk) | ✅ Live |
| Privacy policy URL | ✅ Live at `/privacy` |
| Terms of service URL | ✅ Live at `/terms` |
| Screenshot/demo of Strava data in the app | ✅ Included in email |

---

## Will They Approve Based on What We Have?

**Getting closer.** Strava specifically looks for:

- ✅ A **privacy policy page** explaining how Strava data is stored, used, and deleted — `/privacy` now covers this
- ✅ **"Powered by Strava"** attribution on Settings page — added
- ✅ **Clear explanation of what data is accessed and why** — covered in privacy policy
- ⏳ **Strava brand guidelines compliance** — need official badge assets (currently using a text fallback)

---

## Remaining Steps for Approval

### 1. Complete the Strava API App Settings
At [strava.com/settings/api](https://www.strava.com/settings/api):
- ✅ Upload the square app icon (Goals Club logo)
- ⬜ Write the app description (what it does, how it uses Strava data)
- ⬜ Set website URL to `https://dev.thegoalsclub.co.uk`
- ⬜ Set privacy policy URL to `https://dev.thegoalsclub.co.uk/privacy`
- ⬜ Set terms of service URL to `https://dev.thegoalsclub.co.uk/terms`

### 2. Use Official Strava Badge Assets
Download from [developers.strava.com/guidelines](https://developers.strava.com/guidelines/):
- ⬜ Replace the text "Powered by Strava" with the official badge image
- ⬜ Add "Powered by Strava" or Strava logo where activity data from Strava is displayed (activity history cards)

### 3. Screenshots/Demo
Capture screenshots showing:
- ⬜ The Strava connect flow (`/settings`)
- ⬜ A goal detail page with the Strava Auto-Sync section and activity history populated from Strava

### 4. Submit for Review
- ⬜ Once all items above are complete, submit via the Strava API settings page

---

## Completed ✅

- ✅ Privacy policy page (`/privacy`) — covers Strava data storage, token handling, disconnect flow
- ✅ Terms of service page (`/terms`) — covers Strava integration section
- ✅ "Powered by Strava" attribution on Settings page
- ✅ "Powered by Strava" attribution on Goal Detail (Strava Auto-Sync panel)
- ✅ Privacy/Terms/Contact footer links on all authenticated pages (AppLayout)
- ✅ Privacy and Terms pages rendered within site layout (header + footer)
- ✅ Routes wired: `/privacy`, `/terms`, `/contact` (within AppLayout)
- ✅ Contact page (`/contact`) — mailto link to support@thegoalsclub.co.uk
- ✅ Goals Club logo created and added to site header (with scroll-shrink animation)
- ✅ Logo uploaded to Strava API app settings as app icon

---

## Notes

- An about page and help section are nice but **will not affect approval**
- The **privacy policy was the single most critical item** — now complete
- Test athletes must be explicitly added in dev mode (up to 15)
- Once approved, the 15-athlete limit is removed and any Strava user can connect

---

## Future: Expanded Strava Data Access

Currently we only access activity type, distance, and date (`activity:read` scope). This keeps the approval process simple and the privacy policy lightweight.

**Planned for a future sprint** (post-approval):

| Data | Scope Needed | Use Case |
|---|---|---|
| GPS routes | `activity:read` (already have) | Activity maps on goal pages, route heatmaps |
| Heart rate | `activity:read` (already have) | Richer activity cards (avg/max HR), training insights |
| Photos | `activity:read` (already have) | Auto-attach Strava photos to activity history |
| Private activities | `activity:read_all` | Include non-public Strava activities in goal sync |

**Steps when ready to expand:**
1. Update the Strava OAuth URL to request `activity:read_all` if needed
2. Update `Privacy.tsx` — remove "We do not access heart rate, GPS routes, photos" line
3. Add new fields to the webhook Lambda's activity processing
4. Existing users will need to re-authorize (new OAuth consent) if scope changes
5. No Strava re-approval needed — scope changes don't trigger re-review for approved apps
