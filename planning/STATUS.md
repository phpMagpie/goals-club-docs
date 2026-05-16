# Project Status

**Last Updated:** May 16, 2026

---

## 📍 Current Phase: Soft Launch (Friends Beta) — Dual Environment

Live at both `thegoalsclub.co.uk` (prod) and `dev.thegoalsclub.co.uk` (dev). Strava approved for 999 athletes. All pre-launch priorities complete. Production stack deployed.

---

## ✅ Feature Summary

| Area | Status | Notes |
|------|--------|-------|
| **Auth** | ✅ Complete | Cognito OAuth, username setup modal, self-healing user creation |
| **Dashboard** | ✅ Complete | Goals, events, activity feed, popular goals, getting-started CTA |
| **Goals** | ✅ Complete | Create, join, leave, rejoin, item-based + recurring, progress tracking |
| **Explore** | ✅ Complete | Server-side pagination, search, category/type filters, scroll restoration, past event goals hidden |
| **Events** | ✅ Complete | Browse, filter, "I'm Doing This" joins canonical goal, bidirectional nav |
| **Badges** | ✅ Functional | 67 badges, auto-awarding, confetti. l1/l2 images live, l3-l5 placeholder |
| **Social** | ✅ Complete | Follow/unfollow, reactions, activity feed (ALL/FOLLOWING/MINE), follower modal |
| **Profiles** | ✅ Complete | Public profiles, share button, visibility controls |
| **Strava** | ✅ Complete | OAuth, webhook (SQS-decoupled, prod-only), auto-sync, token refresh, rate limit hardening, activity claiming, route maps |
| **Admin** | ✅ Complete | Full CRUD for all models, event approval workflow, user suspend |
| **Mobile** | ✅ Tested | Dashboard, Explore, Goal Detail, Profile all responsive |
| **Error handling** | ✅ Complete | RDS cold-start auto-retry (3x), friendly beta error message, ErrorAlert component, 404 page, skeleton loaders |
| **Legal** | ✅ Complete | Privacy policy, terms of service, Strava attribution |
| **Multi-env** | ✅ Complete | Dev + prod stacks, `make deploy_dev` / `make deploy_prod`, per-env configs |

---

## 🏗️ Infrastructure

| Component | Service | Notes |
|-----------|---------|-------|
| Database | Aurora Serverless v2 MySQL | Scales to zero, ~5 min auto-pause |
| API | AppSync GraphQL + RDS Data Source | 70+ unit resolvers, 5 pipeline resolvers |
| Auth | Cognito User Pool | OAuth 2.0, per-stack |
| Web frontend | S3 + CloudFront | `thegoalsclub.co.uk` (prod), `dev.thegoalsclub.co.uk` (dev) |
| Admin | S3 + CloudFront | `admin.thegoalsclub.co.uk` (prod), `dev-admin.thegoalsclub.co.uk` (dev) |
| Strava webhook | API Gateway → Lambda → SQS → Processor Lambda | DLQ after 5 retries, prod-only subscription |
| Token refresh | Lambda + EventBridge (every 5 hours) | 6-hour look-ahead |
| Email | SES (DKIM configured) | Welcome email template (per-env) |
| DNS | Route 53 | Wildcard SSL via ACM |
| Monitoring | CloudWatch EMF metrics | Strava rate limits (alarms TBD) |
| IaC | Pulumi (S3 state backend) | `setup.ts` + `.env.prod`/`.secrets.prod` per repo |

---

## 💰 Monthly Cost (dev + prod combined, idle)

| Resource | Cost |
|----------|------|
| Aurora Serverless v2 (×2) | ~$0 (scales to zero) |
| CloudFront (4 distributions) | ~$4 |
| Everything else | Free tier |
| **Total** | **~$10–15/mo** |

---

## 📚 Documentation

See [NEXT_STEPS.md](./NEXT_STEPS.md) for roadmap and all reference doc links.
