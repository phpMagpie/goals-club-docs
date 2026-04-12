# Documentation Review & Consolidation Plan

**Date:** March 8, 2026

## Current Documentation Inventory

| Document | Lines | Purpose | Status |
|----------|-------|---------|--------|
| `PROJECT_OVERVIEW.md` | 220 | Vision, user types, decisions, features | 📕 Dated (Feb 2025) |
| `MVP_SCOPE.md` | 300 | MVP feature definitions | 📕 Dated, superseded |
| `PROJECT_STATUS.md` | 184 | Current state, completed features | ✅ Active |
| `NEXT_STEPS.md` | 485 | Roadmap, weekly tasks, tech debt | ✅ Active |
| `TECHNICAL_ARCHITECTURE.md` | 426 | Infra patterns, repo structure | ✅ Reference |
| `DATABASE_SCHEMA.md` | 520 | Full schema documentation | ✅ Reference |
| `APPSYNC_JS_RUNTIME.md` | 300 | Resolver runtime guidelines | ✅ Reference |
| `MAPS_STRATEGY.md` | 276 | Maps approach for MVP | 📕 Mostly superseded |
| `MAP_VIEW_SETUP.md` | 77 | Mapbox setup instructions | ✅ Reference |
| `DATA_SOURCES.md` | 50 | NT data sourcing notes | 📕 Historical |
| `SEED_ACTIVITIES.md` | 60 | Test data to seed | ⚠️ Low value |
| `SSL_CERTIFICATE_SETUP.md` | 57 | Cert config reference | ✅ Reference |
| `schema.dbml` | N/A | Visual ERD source | ✅ Reference |
| `sessions/SESSION_2026-03-04.md` | 182 | Dev session log | 📕 Historical |
| `sessions/SESSION_2026-03-05.md` | 127 | Dev session log | 📕 Historical |

---

## 🔴 Identified Duplication

### 1. PROJECT_OVERVIEW.md ↔ MVP_SCOPE.md

**Overlap:** Both describe features, user types, badges, social features, organisers

| Content | PROJECT_OVERVIEW | MVP_SCOPE |
|---------|------------------|-----------|
| User types | ✅ | ✅ (partial) |
| Goal categories | ✅ | ✅ (as "MVP Categories") |
| Badge system | ✅ | ✅ |
| Organiser features | ✅ | ✅ |
| Reaction types | ✅ | ✅ (implied) |
| Revenue model | ✅ | ❌ |
| MVP boundaries | ❌ | ✅ |
| Success metrics | ❌ | ✅ |

**Recommendation:** Merge into single `PROJECT.md`

---

### 2. PROJECT_STATUS.md ↔ NEXT_STEPS.md

**Overlap:** Both track what's done, in-progress, and planned

| Content | PROJECT_STATUS | NEXT_STEPS |
|---------|----------------|------------|
| Completed features | ✅ | ✅ (in roadmap) |
| Current phase | ✅ | ✅ |
| Next priorities | ✅ | ✅ (detailed) |
| Tech debt | ❌ | ✅ |
| File references | ✅ | ❌ |
| Resolver structure | ✅ | ❌ |

**Recommendation:** Merge into single `STATUS.md`

---

### 3. MAPS_STRATEGY.md ↔ MAP_VIEW_SETUP.md

**Overlap:** Both cover map implementation

| Content | MAPS_STRATEGY | MAP_VIEW_SETUP |
|---------|---------------|----------------|
| Provider comparison | ✅ | ✅ (brief) |
| Implementation options | ✅ | ❌ |
| Mapbox setup | ✅ | ✅ |
| Component usage | ❌ | ✅ |
| Future phases | ✅ | ❌ |

**Recommendation:** `MAP_VIEW_SETUP.md` is sufficient; archive `MAPS_STRATEGY.md`

---

### 4. Session Logs (sessions/*.md)

**Issue:** Point-in-time logs duplicating info now in STATUS/NEXT_STEPS

**Recommendation:** Archive to `sessions/archive/` - useful for history but not active reference

---

### 5. DATA_SOURCES.md

**Issue:** Historical note about NT data sourcing; no longer actively needed

**Recommendation:** Archive - content outdated (we have 548 NT locations now)

---

### 6. SEED_ACTIVITIES.md

**Issue:** Contains 1 activity (Catbells) - very low value

**Recommendation:** Delete or merge into seeder code comments

---

## 📐 Proposed Restructure

### New Structure

```
docs/planning/
├── README.md                    # Index linking to all docs
├── PROJECT.md                   # Vision + MVP scope + revenue (merged)
├── STATUS.md                    # Current state + roadmap (merged)  
├── TECHNICAL_ARCHITECTURE.md    # Infrastructure patterns (keep)
├── DATABASE_SCHEMA.md           # Schema reference (keep)
├── APPSYNC_JS_RUNTIME.md        # Resolver guidelines (keep)
├── MAP_VIEW_SETUP.md            # Mapbox setup (keep)
├── SSL_CERTIFICATE_SETUP.md     # Cert reference (keep)
├── schema.dbml                  # ERD source (keep)
└── archive/                     # Historical docs
    ├── PROJECT_OVERVIEW.md      # Original vision doc
    ├── MVP_SCOPE.md             # Original MVP doc
    ├── MAPS_STRATEGY.md         # Strategy decisions
    ├── DATA_SOURCES.md          # Data sourcing notes
    ├── SEED_ACTIVITIES.md       # Test data notes
    └── sessions/
        ├── SESSION_2026-03-04.md
        └── SESSION_2026-03-05.md
```

### Document Count

| Current | Proposed |
|---------|----------|
| 14 files | 8 active + 7 archived |

---

## 📝 Merge Plans

### Merge 1: PROJECT.md

Combine `PROJECT_OVERVIEW.md` + `MVP_SCOPE.md`:

**Structure:**
1. Vision & Mission
2. User Types
3. Goal Categories & Types
4. Core Features (from MVP_SCOPE)
5. Social & Community
6. Badges & Rewards
7. Organisers & Events
8. Revenue Model
9. Success Metrics
10. Post-MVP Features (explicitly deferred)

---

### Merge 2: STATUS.md

Combine `PROJECT_STATUS.md` + `NEXT_STEPS.md`:

**Structure:**
1. Current Phase Summary
2. Completed Features (checklist)
3. Current Sprint / This Week
4. Upcoming Priorities
5. Roadmap (weekly breakdown)
6. Technical Debt
7. AppSync Resolver Notes → Link to APPSYNC_JS_RUNTIME.md
8. File References

---

## ✅ Action Items

- [ ] Create `docs/planning/archive/` directory
- [ ] Move historical docs to archive
- [ ] Create merged `PROJECT.md`
- [ ] Create merged `STATUS.md`
- [ ] Delete original files after merge
- [ ] Create `README.md` index
- [ ] Update any cross-references in remaining docs

---

## Decision Required

**Option A: Merge & Archive** (Recommended)
- Clean, organized structure
- Fewer files to maintain
- Clear separation of reference vs active docs

**Option B: Keep All, Add Index**
- Add README.md linking everything
- Mark status on each doc
- Lower effort but still cluttered

**Option C: Minimal Cleanup**
- Just archive session logs and DATA_SOURCES.md
- Keep rest as-is
- Lowest effort

