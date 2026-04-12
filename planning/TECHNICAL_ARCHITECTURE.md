# The Goals Club - Technical Architecture

## Overview
This document outlines the technical architecture for The Goals Club platform, aligned with the proven Dataworks infrastructure patterns.

---

## Architecture Summary

Based on analysis of the Dataworks repos, The Goals Club will follow the same battle-tested patterns:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CloudFront CDN                                │
│                    (thegoalsclub.co.uk + subdomains)                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
    ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
    │   S3 + CDN       │  │   S3 + CDN       │  │    AppSync       │
    │   (Web App)      │  │   (Admin App)    │  │  GraphQL API     │
    │   Public Users   │  │   Refine+Mantine │  │  + Cognito Auth  │
    └──────────────────┘  └──────────────────┘  └──────────────────┘
                                                         │
                                    ┌────────────────────┼────────────────────┐
                                    ▼                    ▼                    ▼
                          ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
                          │  Aurora MySQL   │  │       S3        │  │      SES        │
                          │  Serverless v2  │  │  (Images/Files) │  │    (Email)      │
                          └─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

## Decisions Confirmed ✅

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Database** | Aurora Serverless v2 | Scale to zero, cost effective pre-launch |
| **API** | AppSync (GraphQL) | Real-time subscriptions for activity feeds |
| **Authentication** | Cognito | AWS-native, free tier, proven in Dataworks |
| **Domain** | thegoalsclub.co.uk | Owned and ready |
| **Repository Structure** | Multi-repo | Matches Dataworks pattern |
| **Pulumi State** | S3 backend | Cost-effective, proven pattern |
| **Package Manager** | Bun | Consistent with Dataworks |
| **Payments** | Stripe | Industry standard |
| **Email** | AWS SES | Cost-effective, AWS-native |

---

## Repository Structure (Aligned with Dataworks)

Based on the Dataworks pattern (Data/Console/SDK → Data/Admin/Web/Shared):

```
github.com/your-org/
│
├── goals-club-data/              # API + Infrastructure (like Dataworks-Data)
│   ├── packages/
│   │   ├── infra/                # Pulumi infrastructure
│   │   │   ├── index.ts          # Main exports + env file generation
│   │   │   ├── modules/          # Infrastructure modules
│   │   │   │   ├── appsync-rds/  # AppSync + Aurora config
│   │   │   │   │   ├── appsync-rds.ts
│   │   │   │   │   ├── resolvers/
│   │   │   │   │   └── schema.graphql
│   │   │   │   ├── cognito/      # Auth setup
│   │   │   │   ├── constants.ts
│   │   │   │   ├── databases/
│   │   │   │   ├── s3.ts
│   │   │   │   ├── ses.ts
│   │   │   │   └── shared.ts     # Stack references
│   │   │   ├── utils/
│   │   │   ├── Pulumi.yaml
│   │   │   ├── Pulumi.dev.yaml
│   │   │   └── Pulumi.prod.yaml
│   │   ├── shared/               # Shared within this repo
│   │   │   ├── interfaces/
│   │   │   ├── helpers/
│   │   │   └── services/
│   │   └── tools/
│   │       └── pulumi-docker/    # Docker image for Pulumi runs
│   ├── scripts/
│   ├── test/
│   │   └── bruno/                # API tests
│   ├── Makefile                  # Build/deploy orchestration
│   ├── setup.ts                  # Config loader from .env/.secrets
│   ├── package.json
│   └── tsconfig.json
│
├── goals-club-admin/             # Admin Panel (like Dataworks-Console)
│   ├── packages/
│   │   ├── infra/                # S3 + CloudFront deployment
│   │   │   ├── index.ts
│   │   │   ├── modules/
│   │   │   │   ├── admin-frontend.ts
│   │   │   │   ├── constants.ts
│   │   │   │   └── shared.ts     # Stack refs to goals-club-data
│   │   │   ├── Pulumi.yaml
│   │   │   └── Pulumi.dev.yaml
│   │   └── ui/                   # React + Refine + Mantine
│   │       ├── src/
│   │       │   ├── App.tsx
│   │       │   ├── components/
│   │       │   │   └── custom/   # Custom Mantine wrappers
│   │       │   ├── pages/
│   │       │   ├── providers/
│   │       │   ├── hooks/
│   │       │   └── themes/
│   │       ├── package.json
│   │       └── vite.config.ts
│   ├── Makefile
│   └── package.json
│
├── goals-club-web/               # Public Frontend (like Dataworks-Visual)
│   ├── packages/
│   │   ├── infra/
│   │   │   ├── index.ts
│   │   │   ├── modules/
│   │   │   │   ├── web-frontend.ts
│   │   │   │   └── shared.ts
│   │   │   └── Pulumi.yaml
│   │   └── ui/
│   │       ├── src/
│   │       │   ├── App.tsx
│   │       │   ├── components/
│   │       │   ├── pages/
│   │       │   └── hooks/
│   │       ├── package.json
│   │       └── vite.config.ts
│   ├── Makefile
│   └── package.json
│
└── goals-club-shared/            # Shared SDK (like Dataworks-SDK)
    ├── src/
    │   ├── index.ts              # Main exports
    │   ├── types/                # TypeScript interfaces
    │   │   ├── user.ts
    │   │   ├── goal.ts
    │   │   ├── activity.ts
    │   │   └── index.ts
    │   ├── graphql/              # Generated from schema
    │   ├── helpers/
    │   │   └── writeEnvFiles.ts  # From Dataworks-SDK pattern
    │   ├── constants/
    │   │   ├── categories.ts
    │   │   ├── reactions.ts
    │   │   └── index.ts
    │   ├── validation/           # Zod schemas
    │   │   ├── goal.ts
    │   │   └── activity.ts
    │   └── utils/
    ├── scripts/
    │   └── publish.js
    ├── package.json
    └── tsconfig.json
```

---

## Key Patterns from Dataworks

### 1. Makefile Orchestration

Each repo has a Makefile with consistent targets:

```makefile
# Common variables
PULUMI_S3_ACCESS_TOKEN := "dummy-token-for-s3-backend"
CURRENT_VERSION_FILE := packages/tools/pulumi-docker/.currentversion
PULUMI_DOCKER_IMAGE := your-ecr/pulumi-multiarch:$(IMAGE_VERSION)

# Common targets
default: root build

deploy: default
    @$(MAKE) initialise_stack
    $(call check_credentials,Deploying to,up --yes --skip-preview)
    @$(MAKE) backup_stack

preview: default
    $(call check_credentials,Previewing changes to,preview)

down:
    $(call check_credentials,Downing,down --yes --skip-preview)

backup_stack:
    @cd packages/infra && pulumi stack export --file stacks_backup/stack-$$(pulumi stack --show-name).json.bak

tests:
    bun run test
    bun run test:bruno

clean:
    find . -name 'node_modules' -type d -prune -exec rm -rf '{}' +
```

### 2. Pulumi Docker Container

Pulumi runs inside a versioned Docker container:
- Stored in ECR with version tags
- Ensures consistent environment
- Mounts project, passes credentials
- Works identically locally and in CI

### 3. S3 State Backend

```bash
# Login flow in Makefile:
# 1. Login to Pulumi Cloud to get state bucket URL
pulumi login https://api.pulumi.com
state_bucket=$(pulumi stack output stateBackendUrl --stack org/infra-state/stack)

# 2. Switch to S3 backend
export PULUMI_BACKEND_URL="$state_bucket&awssdk=v2"
pulumi login

# 3. Use passphrase for encryption
export PULUMI_CONFIG_PASSPHRASE=$(op read "op://Vault/item/field")
```

### 4. Stack References (Cross-repo)

```typescript
// In goals-club-admin/packages/infra/modules/shared.ts
import * as pulumi from "@pulumi/pulumi";
import { DATA_INFRA_S3 } from "./constants";

const dataEngine = new pulumi.StackReference(DATA_INFRA_S3);

export const cognitoUserPoolId = dataEngine.getOutput("cognitoUserPoolIdOutput");
export const appSyncApiEndpoint = dataEngine.getOutput("graphqlEndpointOutput");
export const cognitoDomain = dataEngine.getOutput("cognitoDomainOutput");
```

### 5. Environment File Generation

```typescript
// In infra/index.ts - generates .env for frontend
import { writeEnvFile } from "@goals-club/shared";

pulumi.all([cognitoDomain, clientId, apiEndpoint]).apply(([domain, clientId, api]) => {
  writeEnvFile(path.join(__dirname, "../../ui/.env"), [
    { key: "VITE_COGNITO_DOMAIN", value: domain },
    { key: "VITE_COGNITO_CLIENT_ID", value: clientId },
    { key: "VITE_GRAPHQL_API_URL", value: api },
  ]);
});
```

### 6. Bun Workspaces + Conventional Commits

```json
// Root package.json
{
  "workspaces": ["packages/*"],
  "scripts": {
    "prepare": "husky",
    "preinstall": "bun sdk",
    "format": "prettier --write ."
  },
  "lint-staged": {
    "*.{js,ts,tsx}": ["prettier --write"]
  }
}
```

---

## AWS Services

| Service | Purpose |
|---------|---------|
| **AppSync** | GraphQL API with real-time subscriptions |
| **Aurora Serverless v2** | MySQL database (scales to zero) |
| **Cognito** | User authentication |
| **S3** | Static hosting, image storage |
| **CloudFront** | CDN for web/admin apps |
| **SES** | Transactional email |
| **API Gateway** | Stripe webhooks |
| **EventBridge** | Badge award events, async workflows |
| **SQS** | Email queue, image processing |
| **Secrets Manager** | API keys, DB credentials |
| **CloudWatch** | Logging, monitoring |

---

## AppSync Resolvers

Resolvers use the **AppSync JavaScript (APPSYNC_JS) Runtime** with Aurora MySQL/RDS Data Source.

**📖 Full documentation:** [`APPSYNC_JS_RUNTIME.md`](./APPSYNC_JS_RUNTIME.md)

### Resolver Types

| Type | Use Case | Max SQL Statements |
|------|----------|-------------------|
| **Standard Resolver** | Simple CRUD operations | 2 |
| **Pipeline Resolver** | Multi-statement operations, complex logic | Unlimited (1 per function) |

### File Structure

```
packages/infra/modules/appsync-rds/
├── schema.graphql              # GraphQL schema
├── resolvers/                  # Standard resolvers (1-2 SQL statements)
│   └── src/
│       ├── goals/
│       ├── users/
│       └── activities/
├── pipeline-functions/         # Reusable pipeline functions
│   └── src/
│       ├── insertActivity.js
│       └── upsertGoalPeriod.js
└── pipeline-resolvers/         # Pipeline resolver definitions
    └── src/
        └── logGoalActivity.js
```

### Key Constraints

⚠️ The AppSync JS runtime is NOT Node.js. Key restrictions:
- No `for` loops (use `map`, `forEach`, `filter`)
- No `++`/`--` operators (use `+= 1`)
- Max 2 SQL statements per standard resolver
- MySQL DATETIME format: `YYYY-MM-DD HH:MM:SS` (not ISO 8601)

---

## Database Schema

Key tables (see DATABASE_SCHEMA.md for full details):

- **users** - Profile data (Cognito handles auth)
- **goals** - User goals with visibility/join settings
- **goal_participants** - Users who joined shared goals
- **activities** - Progress logs with optional location
- **activity_photos** - Photos attached to activities
- **reactions** - Encouragement reactions on activities
- **follows** - User follows
- **goal_follows** - Goal follows
- **badges** - Virtual badge definitions
- **user_badges** - Earned badges
- **categories** - Goal categories
- **events** - Organiser events
- **organisers** - Event organiser applications
- **products** - Physical merchandise
- **orders** / **order_items** - E-commerce
- **reaction_types** - Configurable reaction options

---

## Frontend Stack

### Admin (Refine + Mantine)
| Library | Purpose |
|---------|---------|
| React 18 | UI framework |
| Vite 7 | Build tool |
| Refine 4 | Admin CRUD framework |
| Mantine 8 | UI components |
| aws-amplify 6 | Cognito auth |
| graphql-request | GraphQL client |

### Web (Public)
| Library | Purpose |
|---------|---------|
| React 18 | UI framework |
| Vite 7 | Build tool |
| Mantine 8 | UI components |
| Leaflet | Maps |
| aws-amplify 6 | Optional auth |

---

## Cost Optimization

### Pre-Launch (~$1-5/month)
- Aurora Serverless v2 scales to zero
- Lambda/AppSync/Cognito free tiers
- Minimal S3 storage

### Post-Launch 1k users (~$50-100/month)
- Aurora: ~$30-50
- Lambda/AppSync: ~$15
- S3/CloudFront: ~$10
- SES: ~$1

---

## Shared Package Publishing

### Option 1: GitHub Packages (Recommended)
```json
{
  "name": "@goals-club/shared",
  "publishConfig": { "registry": "https://npm.pkg.github.com" }
}
```

### Option 2: AWS CodeArtifact (Like Dataworks)
Same auth pattern as Dataworks-SDK.

### Option 3: Local File Reference (Dev)
```json
{ "dependencies": { "@goals-club/shared": "file:../goals-club-shared" } }
```

---

## Next Steps

1. **Create goals-club-shared repo** - Types, helpers, constants
2. **Create goals-club-data repo** - Scaffold from Dataworks-Data
3. **Set up Pulumi state bucket** - New or share Dataworks
4. **Create Cognito setup** - User pool, app clients
5. **Create GraphQL schema** - Core types
6. **Create goals-club-admin repo** - From Dataworks-Console
7. **Create goals-club-web repo** - Public frontend

---

*Document created: February 28, 2025*

