# The Goals Club — Technical Architecture

> Last updated: 2026-05-16

## Overview

The Goals Club is a sports goal tracking platform built on AWS serverless infrastructure, following patterns proven in the Dataworks platform.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             CloudFront CDN                                  │
│              (thegoalsclub.co.uk / dev.thegoalsclub.co.uk)                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
    ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
    │   S3 + CDN       │  │   S3 + CDN       │  │    AppSync       │
    │   (Web App)      │  │   (Admin App)    │  │  GraphQL API     │
    │   React+Mantine  │  │  Refine+Mantine  │  │  + Cognito Auth  │
    └──────────────────┘  └──────────────────┘  └──────────────────┘
                                                         │
                              ┌───────────────────────────┼───────────────────┐
                              ▼                           ▼                   ▼
                    ┌─────────────────┐         ┌─────────────────┐  ┌──────────────┐
                    │  Aurora MySQL   │         │       S3        │  │     SES      │
                    │  Serverless v2  │         │  (Images/Files) │  │   (Email)    │
                    └─────────────────┘         └─────────────────┘  └──────────────┘

                          Strava Integration
    ┌─────────────────────────────────────────────────────────────────┐
    │  Strava API ──webhook──▶ API Gateway ──▶ Receiver Lambda       │
    │                              (POST /strava)    │               │
    │                                                ▼               │
    │                                         ┌────────────┐        │
    │                                         │    SQS     │        │
    │                                         │  + DLQ     │        │
    │                                         └─────┬──────┘        │
    │                                               ▼               │
    │                                     Processor Lambda           │
    │                                     (fetch activity,           │
    │                                      log to goals)             │
    │                                                                │
    │  EventBridge (every 5h) ──▶ Token Refresh Lambda               │
    │  OAuth Lambda (connectStrava mutation) ──▶ RDS Data API        │
    └─────────────────────────────────────────────────────────────────┘
```

---

## Technology Choices

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Database** | Aurora Serverless v2 MySQL | Scale to zero, cost effective |
| **API** | AppSync (GraphQL) | Real-time subscriptions, managed resolvers |
| **Authentication** | Cognito | AWS-native, free tier, per-stack isolation |
| **Frontends** | React 18 + Vite + Mantine 8 | Consistent across web and admin |
| **Admin framework** | Refine 4 | CRUD generation, data provider hooks |
| **Domain** | thegoalsclub.co.uk | Route 53, wildcard SSL via ACM |
| **IaC** | Pulumi (TypeScript) | S3 state backend, stack-per-environment |
| **Package manager** | Bun | Consistent with Dataworks |
| **Payments** | Stripe | Industry standard (future) |
| **Email** | AWS SES | DKIM configured, welcome template |
| **Fitness sync** | Strava API | OAuth + webhook, 999 athletes approved |

---

## Repository Structure

```
goals-club-data/                  # Backend — API, database, Strava integration
├── packages/
│   ├── database/                 # Sequelize migrations + seeders
│   │   ├── migrations/           # Squashed schema + incremental
│   │   ├── seeders/              # Production seed data
│   │   └── seeders/test/         # Test data (separate tracking)
│   ├── infra/                    # Pulumi infrastructure
│   │   ├── modules/
│   │   │   ├── appsync-rds/      # AppSync + Aurora + resolvers + schema
│   │   │   ├── cognito/          # User pool + app clients
│   │   │   ├── databases/        # Aurora cluster + VPC
│   │   │   ├── lambda-strava-*   # 4 Strava Lambda modules
│   │   │   ├── lambda-migrations # DB migration Lambda
│   │   │   ├── ses.ts            # Email
│   │   │   └── s3.ts             # Storage
│   │   └── lambdas/
│   │       ├── strava-oauth/     # OAuth token exchange
│   │       ├── strava-webhook/   # Receiver (enqueue to SQS)
│   │       ├── strava-webhook-processor/  # SQS consumer
│   │       └── strava-token-refresh/      # Scheduled refresh
│   └── tools/
├── test/bruno/                   # Bruno API tests
├── Makefile
└── setup.ts

goals-club-web/                   # Public web app
├── packages/
│   ├── infra/                    # S3 + CloudFront + Route 53
│   └── ui/                       # React SPA
│       └── src/
│           ├── pages/            # Dashboard, Explore, Goals, Events, etc.
│           ├── hooks/            # useCurrentUser, usePublicGoals, etc.
│           ├── components/       # UI components
│           ├── providers/        # Auth provider
│           └── lib/              # GraphQL queries, helpers
├── Makefile
└── setup.ts

goals-club-admin/                 # Admin panel
├── packages/
│   ├── infra/                    # S3 + CloudFront
│   └── ui/                       # Refine + Mantine
│       └── src/pages/            # CRUD for all models
├── Makefile
└── setup.ts

goals-club-shared/                # Shared npm package (@goals-club/shared)
└── src/
    ├── types/                    # TypeScript interfaces
    ├── validation/               # Zod schemas
    └── constants/                # Shared constants
```

---

## AWS Services

| Service | Purpose |
|---------|---------|
| **AppSync** | GraphQL API — 70 unit resolvers, 5 pipeline resolvers |
| **Aurora Serverless v2** | MySQL database (26 tables, scales to zero) |
| **Cognito** | OAuth 2.0 authentication (per-stack User Pool) |
| **S3** | Static hosting (web + admin), image storage |
| **CloudFront** | CDN for web + admin apps |
| **API Gateway** | HTTP API for Strava webhook endpoint |
| **Lambda** | Strava OAuth + history sync (120s timeout), webhook receiver, webhook processor, token refresh, DB migrations |
| **SQS** | Strava activity queue + dead-letter queue (5 retries, prod-only webhook) |
| **EventBridge** | Scheduled token refresh (every 5 hours) |
| **SES** | Transactional email (DKIM configured) |
| **Secrets Manager** | Database credentials (accessed via Pulumi env vars, not VPC endpoint) |
| **Route 53** | DNS (wildcard SSL via ACM) |
| **CloudWatch** | Logging, EMF metrics for Strava rate limits |

---

## AppSync Resolvers

Resolvers use the **AppSync JavaScript (APPSYNC_JS) Runtime** with Aurora MySQL via RDS Data Source.

**📖 Full constraints doc:** [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md)

| Type | Count | Use Case | Max SQL per invocation |
|------|-------|----------|----------------------|
| **Unit resolvers** | 70 | Simple CRUD, field-level resolvers | 2 |
| **Pipeline resolvers** | 5 | Multi-step operations | Unlimited (1 per function) |

**Pipeline resolvers:** `logGoalActivity`, `checkAndAwardBadges`, `createEvent`, `updateEvent`, `commitToEvent`

```
packages/infra/modules/appsync-rds/
├── schema.graphql
├── resolvers/src/                # Unit resolvers by domain
│   ├── goals/
│   ├── users/
│   ├── activities/
│   ├── badges/
│   ├── events/
│   ├── reactions/
│   └── strava/
├── pipeline-functions/src/       # Reusable pipeline functions
└── pipeline-resolvers/src/       # Pipeline resolver definitions
```

**Key constraints** (APPSYNC_JS ≠ Node.js):
- No `for` loops (use `map`, `forEach`, `filter`)
- No `++`/`--` operators (use `+= 1`)
- Max 2 SQL statements per unit resolver
- No `parseFloat()` — use SQL casting instead

---

## Strava Integration

| Component | Purpose |
|-----------|---------|
| **OAuth Lambda** | Exchanges auth code for tokens, stores in `strava_tokens`. Router pattern — also handles `syncStravaHistory` (paginated fetch of all past activities, dedup insert). 120s timeout |
| **Webhook Receiver** | Validates Strava subscription (GET), enqueues events to SQS (POST), returns 200 immediately |
| **Webhook Processor** | SQS-triggered — fetches activity from Strava API, captures GPS + route polyline, matches goal links, logs progress. Throws on 429 for SQS retry |
| **Token Refresh** | EventBridge scheduled (every 5h) — refreshes tokens expiring within 6 hours |
| **SQS Queue** | Decouples webhook from processing. DLQ after 5 failed attempts |
| **Claim Resolver** | `listMyStravaActivitiesForItem` — GPS proximity (0-50) + name similarity (0-40) scoring. `claimStravaActivity` — links activity to goal item |

**Rate limit monitoring:** EMF metrics emitted to CloudWatch (`GoalsClub/Strava` namespace) with 80% warning threshold.

---

## Database

26 tables across 8 domains. See [`DATABASE_SCHEMA.md`](./DATABASE_SCHEMA.md) for full schema.

| Domain | Tables |
|--------|--------|
| **Core** | users, goals, goal_items, user_goals, user_goal_activities, user_goal_periods |
| **Lookup** | goal_types, categories, reaction_types |
| **Social** | user_follows, goal_follows, reactions |
| **Badges** | badges, user_badges |
| **Events** | organisers, event_series, events, user_event_interests |
| **Strava** | strava_tokens, strava_goal_links, strava_activities |
| **E-commerce** | products, orders, order_items |
| **System** | notifications, audit_log |

---

## Frontend Stack

### Web App (Public)
| Library | Purpose |
|---------|---------|
| React 18 | UI framework |
| Vite | Build tool |
| Mantine 8 | UI components |
| React Router 7 | Routing |
| aws-amplify 6 | Cognito auth |
| canvas-confetti | Badge award celebrations |
| react-helmet-async | Page titles / SEO |
| mapbox-gl | Route maps for Strava activities |

### Admin App
| Library | Purpose |
|---------|---------|
| React 18 | UI framework |
| Vite | Build tool |
| Refine 4 | Admin CRUD framework |
| Mantine 8 | UI components |
| aws-amplify 6 | Cognito auth |
| graphql-request | GraphQL client |

---

## Infrastructure Patterns

### Makefile Orchestration
Each repo has consistent targets: `make deploy`, `make preview`, `make tests`, `make clean`.

### Pulumi S3 State Backend
Stack state stored in S3 (not Pulumi Cloud). Encryption via passphrase. Stack-per-environment (`dev`, `prod`).

### Cross-Repo Stack References
Web and admin repos reference data stack outputs (Cognito config, AppSync endpoints) via `pulumi.StackReference`.

### Environment File Generation
Pulumi `preview`/`deploy` auto-generates `.env` files for frontends with Cognito domain, client ID, API URLs, Strava client ID.

### Shared Package
`@goals-club/shared` published to npm (GitHub Packages). Contains TypeScript interfaces, Zod validation schemas, and shared constants.

---

## Cost (dev + prod combined, idle)

| Resource | Monthly |
|----------|---------|
| Aurora Serverless v2 (×2) | ~$0 (scales to zero) |
| CloudFront (4 distributions) | ~$4 |
| Everything else | Free tier |
| **Total** | **~$10–15/mo** |

At 1,000 users: ~$50–100/mo (Aurora ~$30–50, Lambda/AppSync ~$15, CDN ~$10).

---

## Multi-Environment Support

The infrastructure supports multiple stacks (`dev`, `prod`) with parameterised resource names, subdomains, and database names. See [`PROD_STACK_DEPLOYMENT.md`](./PROD_STACK_DEPLOYMENT.md) for deployment guide.

| Stack | Web URL | Admin URL | Database |
|-------|---------|-----------|----------|
| dev | dev.thegoalsclub.co.uk | dev-admin.thegoalsclub.co.uk | goalsclub_dev |
| prod | thegoalsclub.co.uk | admin.thegoalsclub.co.uk | goalsclub_prod |
