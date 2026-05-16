# Strava API Approval

## Current Status: ✅ Approved

**Approved:** 2026-05-10 — 999 athlete connections available.

The Strava integration is live and working. All users can connect their Strava accounts.

---

## Limits

| Limit | Value |
|-------|-------|
| Connected athletes | 999 |
| Overall rate limit | 600 requests / 15 min, 6,000 / day |
| Read rate limit | 300 requests / 15 min, 3,000 / day |

Rate limit hardening is deployed — see [STRAVA_RATE_LIMITS.md](./STRAVA_RATE_LIMITS.md) for full analysis.

---

## Compliance Checklist (all complete)

| Requirement | Status |
|---|---|
| App icon (square image) | ✅ Goals Club logo |
| App description | ✅ |
| Website URL | ✅ dev.thegoalsclub.co.uk |
| Privacy policy URL | ✅ `/privacy` |
| Terms of service URL | ✅ `/terms` |
| "Powered by Strava" attribution | ✅ Settings page + Goal Detail |
| Privacy policy covers Strava data | ✅ Storage, tokens, disconnect flow |

---

## Future: Expanded Strava Data Access

Currently we only access activity type, distance, and date (`activity:read` scope).

| Data | Scope Needed | Use Case |
|---|---|---|
| GPS routes | `activity:read` (already have) | Activity maps, route heatmaps |
| Heart rate | `activity:read` (already have) | Richer activity cards, training insights |
| Photos | `activity:read` (already have) | Auto-attach Strava photos to activity history |
| Private activities | `activity:read_all` | Include non-public Strava activities in goal sync |

**Steps when ready to expand:**
1. Update the Strava OAuth URL to request `activity:read_all` if needed
2. Update `Privacy.tsx` — remove "We do not access heart rate, GPS routes, photos" line
3. Add new fields to the webhook Lambda's activity processing
4. Existing users will need to re-authorize (new OAuth consent) if scope changes
5. No Strava re-approval needed — scope changes don't trigger re-review for approved apps
