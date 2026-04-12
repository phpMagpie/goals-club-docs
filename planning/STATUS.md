# Project Status & Roadmap

**Last Updated:** March 8, 2026

---

## 📊 Current Phase: Week 4 — Super Goals & Wainwright Regions

### This Week's Focus
- Super goal structure (parent/child goals)
- Wainwright regional split
- NT "Visit All UK" super goal

---

## ✅ Completed Features

### Core Infrastructure ✅
- [x] Aurora Serverless v2 MySQL database
- [x] AppSync GraphQL API with RDS data source
- [x] Cognito authentication (Admin + Web clients)
- [x] S3 + CloudFront deployments (Admin, Web)
- [x] Route 53 DNS (migrated from Cloudflare)
- [x] SES for email (DKIM configured)
- [x] SSL certificate (wildcard *.thegoalsclub.co.uk)

### Admin Console ✅
- [x] Refine + Mantine 8 setup
- [x] CRUD for Goals, Categories, Users
- [x] Goal types management
- [x] Activity log viewing

### Web App ✅
- [x] User authentication (register/login)
- [x] Dashboard with real data
- [x] Goals list (my goals)
- [x] Explore page (browse public goals)
- [x] Join goal functionality
- [x] Goal detail page with item list

### Collection Goals ✅
- [x] Wainwrights (214 peaks) seeded
- [x] National Trust (11 regional goals, 548 sites)
- [x] Item completion tracking
- [x] Progress calculation (items completed / total)
- [x] Multiple visits to same item

### Maps Integration ✅
- [x] Mapbox GL for location-based goals
- [x] Conditional display (only for location-based goals)
- [x] Green markers = completed, Blue = pending
- [x] Popup details with completion status
- [x] Auto-fit bounds to show all markers

### Frequency/Distance Goals ✅
- [x] Target value, unit, frequency fields
- [x] Period tracking (daily/weekly/monthly)
- [x] Pipeline resolver for atomic updates
- [x] `user_goal_periods` table
- [x] Streak tracking (current + longest streak)
- [x] Period history UI with visual indicators
- [x] Example goals: "10,000 Steps a Day", "Cycle to Work 3x per Week"

### Authentication ✅
- [x] Cognito with Admin/Web app clients
- [x] Admin users in "Admins" group
- [x] Self-registration for web users
- [x] Auto-create user record on first login

---

## 🔜 Next Priorities

### Week 4: Super Goals (Paused)
- [ ] Add `parent_goal_id` to goals table
- [ ] Create `super_goal` goal_type
- [ ] "Visit All NT Sites (UK)" super goal (links 11 regional goals)
- [ ] Split Wainwrights into 7 regional goals
- [ ] "Complete All Wainwright Regions" super goal
- [ ] Auto-calculate progress from child goals

### Week 5: Badges & Rewards ✅ COMPLETE
- [x] Badge trigger system (criteria-based checking)
- [x] Auto-award badges via `checkAndAwardBadges` mutation
- [x] "Check for Badges" button on Badges page
- [x] Confetti animation on badge award
- [x] Modal showing newly earned badges
- [x] Initial system badges seeded (Founding Member, Goal Getter, First Steps, etc.)
- [x] Badge display on profile page
- [x] Auto-check badges after logging activity

### Week 6: Social Features
- [ ] Activity feed from followed users/goals
- [ ] Follow/unfollow functionality
- [ ] Reactions/cheers on activities
- [ ] Public profile pages

---

## 🗓️ Roadmap

### MVP (Launch Target)
| Week | Focus | Status |
|------|-------|--------|
| 1 | Foundation & Core Goal Flow | ✅ Complete |
| 2 | Activity Logging & Progress | ✅ Complete |
| 3 | Goal Types & Multiple Visits | ✅ Complete |
| **4** | **Super Goals & Wainwright Regions** | 🔄 In Progress |
| 5 | Badges & Rewards | ⬜ Planned |
| 6 | Social Features | ⬜ Planned |
| 7 | Profile Polish & Launch Prep | ⬜ Planned |

### Post-MVP (After Launch)
| Phase | Focus | Notes |
|-------|-------|-------|
| 2.1 | Physical Merchandise Shop | Need merch partners first |
| 2.2 | Event Organisers | Organiser registration + events |
| 2.3 | Strava Integration | Requires live site for OAuth |
| 2.4 | Local Goal Clubs | Organic community growth needed |

---

## 📁 Project Structure

### Repositories
```
goals-club-data/    # API, Database, Infrastructure
goals-club-admin/   # Admin Console (Refine + Mantine)
goals-club-web/     # Public Web App (Mantine)
goals-club-shared/  # Shared types, helpers, constants
```

### Key Directories (goals-club-data)
```
packages/infra/modules/appsync-rds/
├── schema.graphql              # GraphQL schema
├── resolvers/src/              # Standard resolvers
│   ├── activities/
│   ├── badges/
│   ├── categories/
│   ├── goals/
│   └── users/
├── pipeline-functions/src/     # Pipeline functions
│   ├── insertActivity.js
│   ├── getGoalPeriodInfo.js
│   └── upsertGoalPeriod.js
└── pipeline-resolvers/src/     # Pipeline resolver definitions
    └── logGoalActivity.js
```

---

## 🔧 Technical Notes

### AppSync JS Runtime
**📖 Full guide:** [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md)

Key constraints:
- No `for` loops (use `map`, `forEach`)
- No `++`/`--` operators (use `+= 1`)
- Max 2 SQL statements per resolver (use pipeline resolvers for more)
- MySQL DATETIME format: `YYYY-MM-DD HH:MM:SS`

### Cost Management
- Stacks up: ~$0.30-$5/day
- Stacks down: ~$0.50/month (Route 53 hosted zone only)
- Use `pulumi down` or `pulumi destroy` when not developing
- Aurora auto-pauses after 5 minutes idle

**Idle costs breakdown:**
| Service | Monthly Cost | Notes |
|---------|-------------|-------|
| Route 53 hosted zone | $0.50 | Unavoidable while using AWS DNS — created manually, not managed by Pulumi |
| Secrets Manager | $0.40/secret | Orphaned RDS secrets can linger after destroy — see cleanup below |

**After `pulumi destroy`, always clean up orphaned RDS secrets:**
```bash
AWS_PROFILE=goalsclub aws secretsmanager list-secrets --region eu-west-1 \
  --query "SecretList[*].Name" --output text | tr '\t' '\n' | while read secret; do
  AWS_PROFILE=goalsclub aws secretsmanager delete-secret \
    --secret-id "$secret" --force-delete-without-recovery --region eu-west-1
done
```
Or use the Makefile target: `make cleanup_secrets`

**Actual costs observed:**
| Month | Cost | Notes |
|-------|------|-------|
| March 2026 | $10.31 | Stacks active, multiple destroy/redeploy cycles |
| April 2026 | ~$0.50 | After secret cleanup (Route 53 only) |

### Admin Users
- paul@thegoalsclub.co.uk (Admin)
- lynsey@thegoalsclub.co.uk (Admin)

---

## 🐛 Known Issues

### Build Issue (esbuild)
Occasionally `bun run build` fails with "The service was stopped":

```bash
pkill -9 -f esbuild
pkill -9 -f vite
rm -rf node_modules/.cache node_modules/.vite
bun run build
```

Or use `npx vite build` directly.

---

## 📝 Quick Commands

```bash
# Start dev servers
cd goals-club-web/packages/ui && bun run dev   # Web: http://localhost:5176
cd goals-club-admin/packages/ui && bun run dev # Admin: http://localhost:5175

# Deploy stacks
cd goals-club-data && make deploy
cd goals-club-web && make deploy
cd goals-club-admin && make deploy

# Destroy stacks (save costs)
cd goals-club-data && make down
```

---

## 📚 Documentation Index

| Document | Purpose |
|----------|---------|
| [`PROJECT.md`](./PROJECT.md) | Vision, features, revenue model |
| [`TECHNICAL_ARCHITECTURE.md`](./TECHNICAL_ARCHITECTURE.md) | Infrastructure patterns |
| [`DATABASE_SCHEMA.md`](./DATABASE_SCHEMA.md) | Full schema reference |
| [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md) | Resolver guidelines |
| [`MAP_VIEW_SETUP.md`](./MAP_VIEW_SETUP.md) | Mapbox configuration |
| [`SSL_CERTIFICATE_SETUP.md`](./SSL_CERTIFICATE_SETUP.md) | Certificate reference |

---

## 🏔️ Test Data to Re-seed on Stack Rebuild

| Goal Item | Date | Notes |
|-----------|------|-------|
| Catbells | 12/04/2023 | Really popular, wore brand new Van boots, bad mistake. |

---

## ✅ Completed Log

| Date | Milestone |
|------|-----------|
| 2026-03-01 | Git repos initialized |
| 2026-03-02 | Route 53 DNS migration, Admin user auto-creation |
| 2026-03-03 | Resolver reorganization, First activity logged |
| 2026-03-04 | National Trust 548 locations seeded, Map view added |
| 2026-03-07 | Frequency goals, Streak tracking, Pipeline resolvers |
| 2026-03-08 | Documentation consolidation |
| 2026-03-08 | **Badge system complete** - Auto-award, confetti, modal celebration |

---

*Document consolidated: March 8, 2026*
*Original sources: PROJECT_STATUS.md, NEXT_STEPS.md*

