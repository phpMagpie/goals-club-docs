# Pulumi State Management

**Last Updated:** May 19, 2026

When Pulumi state gets out of sync with AWS (e.g. manually deleting resources from the AWS Console), deploys will fail with `NotFoundException` or similar errors. This doc covers recovery procedures.

---

## 🚨 Common Symptom

```
error: updating AppSync Resolver (...): operation error AppSync: UpdateResolver,
  StatusCode: 404, NotFoundException: No resolver found.
```

This means Pulumi thinks the resource exists (it's in state) but AWS says it doesn't. The resource was deleted from AWS but not from Pulumi state.

---

## 🔧 Fix: Delete Stale Resource from Pulumi State

### 1. Get the URN from the error message

The error includes the full URN:
```
urn:pulumi:prod::goals-club-data::aws:appsync/resolver:Resolver::GoalsClub-prod-deleteGoalActivity-pipeline-resolver
```

### 2. Login to Pulumi and select the stack

```bash
cd packages/infra
export AWS_PROFILE=goalsclub
export PULUMI_CONFIG_PASSPHRASE='<passphrase>'
pulumi login 's3://goalsclub-pulumi-state?region=eu-west-1'
pulumi stack select prod  # or dev
```

**Note:** The S3 URL must be quoted in zsh (the `?` in the query string triggers glob expansion otherwise).

### 3. Delete the stale resource

```bash
pulumi state delete '<full-urn>' --yes
```

### 4. Do the same for other stacks if needed

```bash
pulumi stack select dev
pulumi state delete 'urn:pulumi:dev::goals-club-data::...' --yes
```

### 5. Redeploy

The next deploy will CREATE the resource fresh instead of trying to UPDATE a ghost.

---

## 🔄 When This Happens

| Scenario | What to do |
|----------|-----------|
| Manually deleted a resolver from AWS Console | `pulumi state delete` the old resource |
| Converted unit resolver → pipeline resolver | Delete both the old unit resolver AND the new pipeline resolver from state if either was partially created |
| Failed deploy left a half-created resource | Delete the resource from state, redeploy |
| Resource exists in AWS but not in state | Use `pulumi import` to bring it under management |

---

## ⚠️ Prevention

- **Never manually delete AWS resources** that Pulumi manages. Use `pulumi destroy` or remove from code and redeploy.
- If you must delete from AWS Console (e.g. debugging), immediately run `pulumi state delete` for that resource before the next deploy.
- When converting unit resolvers to pipeline resolvers, the old unit resolver source file should be deleted AND the infra code should handle the transition. Pulumi sees these as different resources (different names: `-resolver` vs `-pipeline-resolver`).

---

## 📚 Useful Commands

```bash
# List all resources in current stack
pulumi stack export | grep urn | head -50

# Search for a specific resource
pulumi stack export | grep 'deleteGoalActivity'

# Delete a resource (interactive confirmation)
pulumi state delete '<urn>'

# Delete without confirmation
pulumi state delete '<urn>' --yes

# Import an existing AWS resource into Pulumi state
pulumi import aws:appsync/resolver:Resolver <resource-name> <api-id>/<type>/<field>
```

