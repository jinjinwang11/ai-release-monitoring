# Understanding `secrets: inherit`

## 🔑 How It Works

When you use `secrets: inherit`, GitHub looks for secrets in this order:

```
Priority 1: Repository-level secrets
    ↓ (if not found)
Priority 2: Organization-level secrets  ✅ YES, IT WORKS!
    ↓ (if not found)
Priority 3: Environment secrets (if environment specified)
```

---

## ✅ Organization-Level Secrets ARE Inherited

**Question:** If secrets are configured at organization level, can `secrets: inherit` access them?

**Answer:** **YES!** This is exactly how Traveloka does it.

---

## 📊 Evidence from Traveloka

### All fprmfdt workflows use `secrets: inherit`:

```yaml
# CI Build
jobs:
  ci:
    uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/ci_build.yaml@v1
    secrets: inherit

# ECS Deployment (DEV)
jobs:
  deploy:
    uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml@v1
    secrets: inherit

# ECS Deployment (PRODUCTION)
jobs:
  deploy:
    uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml@v1
    secrets: inherit

# Rollback
jobs:
  rollback:
    uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/rollback.yaml@v1
    secrets: inherit
```

### These workflows need secrets like:
- AWS credentials (multiple accounts: dev, staging, prod)
- GitHub tokens
- CodeDeploy configurations
- Artifactory credentials
- Etc.

### But we don't see these secrets in fpr-fprmfdt repo!

**Conclusion:** They MUST be configured at **Organization-level** and inherited via `secrets: inherit`.

---

## 🎯 For AI Deployment Reports

### Recommended: Organization-Level (Traveloka Standard)

**Step 1: Create org secrets (Platform/DevOps team)**

Organization Settings → Actions → Secrets:
```
DATADOG_API_KEY
DATADOG_APP_KEY
LARK_WEBHOOK_DEPLOYMENT_REPORTS
```

**Step 2: All services automatically inherit**

```yaml
# fpr-fprmfdt/.github/workflows/ai-deployment-report.yaml
jobs:
  ai-report:
    uses: jinjinwang11/ai-release-monitoring/.github/workflows/deployment-report.yaml@main
    with:
      service_name: fprmfdt
      deployment_time: ${{ inputs.deployment_time }}
      commit_sha: ${{ inputs.commit_sha }}
    secrets: inherit  # ✅ Gets org-level secrets automatically!
```

**Benefits:**
- ✅ Configure once for ALL services (6-20 services)
- ✅ Rotate API key → all services updated automatically
- ✅ Consistent with Traveloka patterns (CI, deployment, rollback)
- ✅ No per-service configuration needed

---

## 📋 Override Pattern

You can override org-level secrets at repo level if needed:

```
Organization-level:
  LARK_WEBHOOK_DEPLOYMENT_REPORTS = "https://...team-channel..."

fpr-fprmfdt repo-level:
  LARK_WEBHOOK_DEPLOYMENT_REPORTS = "https://...fprmfdt-specific..."
  (overrides org-level)

fpr-fprfb repo-level:
  (no override, uses org-level)
```

Workflow uses `secrets: inherit` in both cases - it's automatic!

---

## 🔐 Security

**Organization secrets access control:**

1. **Visibility:** Can be configured for:
   - All repositories
   - Selected repositories only
   - Private repositories only

2. **Permissions:** Only org admins can:
   - Create org-level secrets
   - Modify org-level secrets
   - View org-level secrets (values are always masked)

3. **Audit:** All secret access is logged in org audit log

---

## 📖 GitHub Documentation

**Official docs confirm this:**

> "Using `secrets: inherit` in a called workflow passes all secrets that are available to the caller workflow. This includes organization secrets, repository secrets, and environment secrets."

Source: [GitHub Docs - Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows#passing-secrets-to-nested-reusable-workflows)

---

## ✅ Summary

| Configuration Level | Inherited by `secrets: inherit`? | Use Case |
|-------------------|----------------------------------|----------|
| **Organization** | ✅ YES | Shared secrets across all services (recommended) |
| **Repository** | ✅ YES | Service-specific overrides or PoC testing |
| **Environment** | ✅ YES | Environment-specific secrets (prod vs dev) |

**Traveloka Pattern:**
- Organization-level for shared infrastructure (AWS, GitHub, etc.)
- `secrets: inherit` in all workflows
- Repo-level overrides only when needed

**Our AI Report Pattern:**
- Follow same approach
- Org-level: DATADOG_API_KEY, DATADOG_APP_KEY, LARK_WEBHOOK
- All services use `secrets: inherit`
- Zero per-service configuration

---

## 🚀 Recommendation

**For PoC:**
- Start with repo-level secrets in `ai-release-monitoring` (self-test)
- Validate workflow works

**For Production:**
- Request Platform team to create org-level secrets
- All services automatically inherit
- Zero maintenance per service
