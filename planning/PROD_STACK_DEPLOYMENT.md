# Production Stack Deployment Guide

Created: 2026-05-10
**Status: ✅ COMPLETED — May 16, 2026**

All three repos (data, web, admin) successfully deployed to prod. Both environments running.

---

## Overview

The Goals Club runs on dual `dev` and `prod` stacks. The app is live at `thegoalsclub.co.uk` (prod) and `dev.thegoalsclub.co.uk` (dev).

---

## Current State — Deployed ✅

All infrastructure is fully parameterised by stack name. Deploy to either environment via:

```bash
make deploy_dev    # or just `make deploy`
make deploy_prod
```

### What already works ✅

- **Resource naming** — all resources use `${PROJECT_NAME}` which includes the stack name via `getStackName()`. No naming collisions between stacks.
- **Subdomains** — `stack.ts` handles prod vs dev prefixing:
  - `dev` → `dev.thegoalsclub.co.uk`, `dev-api.thegoalsclub.co.uk`
  - `prod` → `thegoalsclub.co.uk` (root), `api.thegoalsclub.co.uk`
- **Cognito** — per-stack. Each stack creates its own User Pool, clients, and domain (`goalsclub-dev` vs `goalsclub-prod`). No shared Cognito.
- **Cross-stack refs** — web and admin use `dataEnv` config to point at the correct data stack via `shared.ts`. Set `dataEnv=prod` and it references `organization/goals-club-data/prod`.
- **Makefile / setup.ts** — parameterised via `GOALS_CLUB_ENVIRONMENT` env var.
- **DATABASE_NAME constant** — correctly uses `` `goalsclub_${ENV}` `` in `constants.ts`, but it's **not used everywhere** (see fixes below).

---

## Fixes Required Before Prod Deploy

> **All 4 fixes applied — May 15, 2026.** Deploy to dev and verify before creating prod stack.

### 1. ~~Hardcoded `"goalsclub_dev"` Database Name~~ ✅ Fixed

All 6 Pulumi modules now import `DATABASE_NAME` from `constants.ts` (which resolves to `` `goalsclub_${ENV}` ``). All 4 Lambda source files now use `process.env.DB_NAME!` (no fallback — will fail fast if env var is missing).

### 2. ~~SES Email Template Name Collision~~ ✅ Fixed

Template name changed from `"GoalsClub-Welcome"` to `` `GoalsClub-Welcome-${ENV}` ``. Each stack gets its own template.

### 3. ~~SES Domain Identity Collision~~ ✅ Fixed

Route 53 SES verification and DKIM records now wrapped in `if (isProduction())`. The SES domain identity is created per-stack but DNS records only in prod — once verified, all stacks can send.

### 4. ~~Production Safety Flags~~ ✅ Fixed

- `skipFinalSnapshot: !isProduction()` — prod gets a final snapshot on cluster deletion
- `forceDestroy: !isProduction()` — prod S3 buckets can't be accidentally deleted with objects

---

## Strava Webhook — Decision Implemented ✅

Strava only allows **one webhook subscription per API app**. We went with **Option B: Share One Strava App**.

- Webhook subscription is **prod-only** — `strava-webhook-subscription.ts` is wrapped in `isProduction()` check
- Dev deploys no longer steal the webhook from prod
- Dev can still use Strava OAuth + manual sync (Settings → "Sync History")
- Strava app "Authorization Callback Domain" is set to `thegoalsclub.co.uk` (covers all subdomains for OAuth)

---

## Cost Per Stack

| Resource | Monthly Cost (idle) | Notes |
|----------|-------------------|-------|
| Aurora Serverless v2 | ~$0 | Scales to zero with `minCapacity: 0` |
| CloudFront | ~$1 | Minimal traffic at soft launch |
| Cognito | $0 | Free tier covers 50k MAUs |
| AppSync | ~$0 | Pay per request |
| S3 | ~$0.50 | Frontend assets |
| Lambda | ~$0 | Free tier |
| SQS | ~$0 | Free tier |
| **Total** | **~$5–8/mo** | VPC endpoint removed (was ~$15) |

---

## Deploy Steps

### Prerequisites
1. Apply all fixes from section above (hardcoded DB names, SES, safety flags)
2. Decide on Strava approach (Option A or B)
3. Deploy and test fixes on dev first

### Create Prod Stacks

```bash
# 1. Data stack (backend — must be first)
cd goals-club-data
pulumi stack init prod

# Set config
pulumi config set env prod
pulumi config set domain thegoalsclub.co.uk
pulumi config set --secret DATABASE_PASSWORD <new-secure-password>
pulumi config set --secret STRAVA_CLIENT_ID <prod-or-shared-client-id>
pulumi config set --secret STRAVA_CLIENT_SECRET <prod-or-shared-secret>
pulumi config set --secret STRAVA_VERIFY_TOKEN <new-verify-token>

# Deploy (creates Aurora, Cognito, AppSync, Lambdas)
make deploy

# Run migrations
make db_migrate

# Seed data (categories, badges, goal types, events, organisers)
make db_seed

# 2. Web stack (frontend — references data stack)
cd goals-club-web
pulumi stack init prod

# Set config
pulumi config set env prod
pulumi config set dataEnv prod
pulumi config set domain thegoalsclub.co.uk
pulumi config set STRAVA_CLIENT_ID <same-client-id-as-data>

# Deploy (builds UI, uploads to S3, creates CloudFront)
make deploy

# 3. Admin stack (optional — same pattern)
cd goals-club-admin
pulumi stack init prod
pulumi config set env prod
pulumi config set dataEnv prod
pulumi config set domain thegoalsclub.co.uk
make deploy
```

### Post-Deploy

1. **Strava webhook** — re-register pointing at prod API Gateway URL
2. **Update Strava app settings** — add `thegoalsclub.co.uk` as an authorized callback domain
3. **Test full flow** — sign up, create goal, join goal, connect Strava, log activity
4. **DNS propagation** — CloudFront + Route 53 may take up to 15 minutes

### Switching Between Stacks

```bash
# Dev
pulumi stack select dev
make deploy

# Prod
pulumi stack select prod
make deploy
```

---

## Strava Callback Domain

Strava's app settings have an "Authorization Callback Domain" field. Set it to `thegoalsclub.co.uk` (without subdomain prefix) — this covers both:
- `dev.thegoalsclub.co.uk/callback/strava`
- `thegoalsclub.co.uk/callback/strava`

No protocol (`https://`) or path needed — just the domain.

---

## Future: Shared Resources

If running both stacks long-term, consider extracting these into a shared stack (like Dataworks-Shared):
- Route 53 hosted zone
- ACM certificates
- SES domain verification
- Possibly Cognito (if you want single sign-on across environments — not recommended for dev/prod isolation)

For now, the per-stack approach works fine. The only collision risk is SES (addressed in fix #3 above).

