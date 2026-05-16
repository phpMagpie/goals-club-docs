# Scroll Restoration — `useScrollRestoration`

**File:** `goals-club-web/packages/ui/src/hooks/useScrollRestoration.ts`

## Purpose

Preserves and restores a page's vertical scroll position when a user navigates away and returns via the browser back/forward buttons. This prevents the common UX issue where returning to a long list (e.g. Explore) snaps back to the top.

## Prerequisites

`window.history.scrollRestoration = "manual"` **must** be set at app startup (in `index.tsx`) before this hook is used.

Without it, the browser's built-in scroll restoration fires asynchronously on back/forward navigation and races against (and can override) the custom `window.scrollTo` call. This is the primary cause of a silent "no scroll" bug when returning to a page with many rendered items.

## How It Works

The hook uses two `useEffect` calls and React Router's `useNavigationType` to coordinate save and restore behaviour.

### Save — on unmount

When the component unmounts (i.e. the user navigates away), the current `window.scrollY` position is written to `sessionStorage` under the key `scroll_${key}`.

```ts
return () => {
  sessionStorage.setItem(`scroll_${key}`, String(Math.round(window.scrollY)));
};
```

### Restore — on mount after POP navigation

When the hook mounts and the navigation type is `"POP"` (browser back/forward), it reads the saved position from `sessionStorage` and restores it using `window.scrollTo`.

Rather than a fixed double `requestAnimationFrame`, the hook uses a **rAF retry loop** that checks whether the page is actually tall enough to scroll to the saved position before attempting the scroll. It retries each frame up to 30 times (~500 ms at 60 fps).

```ts
const attempt = () => {
  const pageIsReady = document.documentElement.scrollHeight >= y + window.innerHeight;
  if (pageIsReady || attempts >= MAX_ATTEMPTS) {
    window.scrollTo({ top: y, behavior: "instant" });
    return;
  }
  attempts++;
  requestAnimationFrame(attempt);
};
requestAnimationFrame(attempt);
```

**Why this matters:** a large list (e.g. 3 pages of cards in a `SimpleGrid`) may need several layout passes before it reaches full page height. Without the height check, `window.scrollTo(800)` can fire while `document.documentElement.scrollHeight` is still below 800px — causing it to silently clamp to 0 or whatever the current max is.

The `hasRestored` ref prevents the restore logic from firing more than once per mount.

## When Does Scroll Restoration Fire?

**Only when returning via the browser back/forward button** (`navigationType === "POP"`).

| How you leave Explore | How you return | Scroll restored? |
|---|---|---|
| Click any nav link | Browser **back button** | ✅ Yes |
| Join a goal (navigate to /goals) | Browser **back button** | ✅ Yes |
| Click any nav link | Click Explore in the nav bar | ❌ No — fresh PUSH navigation |
| Browser back/forward | Click Explore in the nav bar | ❌ No — fresh PUSH navigation |

Returning via the nav bar is intentionally treated as a fresh visit. Only the hardware/software back button produces a POP navigation.



```ts
useScrollRestoration(key: string, isReady: boolean)
```

| Parameter | Type | Description |
|---|---|---|
| `key` | `string` | Unique identifier for this page's scroll position in `sessionStorage` (e.g. `"explore"`). |
| `isReady` | `boolean` | Set to `true` once the page content has loaded and the DOM has real height. Prevents premature scroll before data renders. |

## Usage Example

```tsx
const { data, isLoading } = useActivities();

useScrollRestoration("explore", !isLoading);
```

## Storage

Uses `sessionStorage` so scroll positions are scoped to the current browser tab and cleared when the tab is closed. This avoids stale positions persisting across separate sessions.

## Known Bug (Fixed — May 2026)

**Symptom:** Loading 3 pages of goals in Explore, scrolling to mid-page, navigating away, then pressing the browser back button — the 3 pages were restored but scroll position was not.

**Root causes (three separate bugs):**

1. **Browser native scroll restoration race.** `history.scrollRestoration` defaults to `"auto"`. On back navigation the browser fires its own scroll restore asynchronously, which overwrote our custom `window.scrollTo` call. Fixed by setting `history.scrollRestoration = "manual"` in `index.tsx`.

2. **Double rAF too early for large grids.** Two animation frames is enough for a small list, but 72 cards in a `SimpleGrid` can need more layout passes before `scrollHeight` reaches the target. `window.scrollTo(800)` fired while the page was only, say, 400px tall — silently clamping to 0. Fixed by replacing the double rAF with a retry loop that checks `scrollHeight >= target + viewport` before scrolling.

3. **React StrictMode double-invoke corrupting both the save and `usePublicGoals`.**

   a. **Save phase:** The unmount-cleanup approach wrote `scrollY = 0` to sessionStorage during StrictMode's fake unmount, before the page had scrolled. Fixed by moving the save to a `scroll` event listener — the fake unmount cleanup just removes the listener and never touches sessionStorage.

   b. **`usePublicGoals` skip logic:** The original `skipFirstFetch` ref was set to `false` in the first effect run. StrictMode's second (fake remount) run saw `false` and executed `setGoals([]) + fetch`, wiping the 72 restored goals and replacing them with 24 (page 1 only) — making the page too short for the saved scroll position. Fixed by replacing the ref flag with a `filterKey` string comparison: skip only when filters haven't changed AND goals already exist. Since React preserves state across the fake unmount/remount cycle, the goals ref stays at 72 on the second run, and the filter key is unchanged — so both runs correctly skip. A real filter change still triggers a proper re-fetch.



## Limitations & Considerations

- Only restores on `"POP"` navigation — intentionally does not restore on `"PUSH"` or `"REPLACE"` (i.e. fresh navigation to the page always starts at the top).
- The `isReady` flag must accurately reflect when the page content has rendered. If passed `true` too early (before async data loads), the page may not have enough height and the scroll will silently fail.
- Scroll position is saved per-key — if the same key is reused across different pages, positions will collide.

