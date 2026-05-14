# Secret Configuration Guide

## 🔑 Traveloka Pattern: Organization-Level Secrets

Based on Traveloka's standard practices (Cursor agent, ECS deployment), we use **`secrets: inherit`** pattern.

---

## ✅ Recommended: Organization-Level Secrets

**This is how Traveloka does it** (same as Cursor agent and bei-backend-ci-cd-platform):

### Setup (One-time, at org level)

1. **Go to GitHub Organization Settings**
   - https://github.com/organizations/YOUR_ORG/settings/secrets/actions

2. **Create Organization Secrets:**

   | Secret Name | Value | Repository Access |
   |-------------|-------|-------------------|
   | `DATADOG_API_KEY` | From Datadog → Organization Settings → API Keys | All repositories |
   | `DATADOG_APP_KEY` | From Datadog → Organization Settings → Application Keys | All repositories |
   | `LARK_WEBHOOK_DEPLOYMENT_REPORTS` | Lark webhook URL | All repositories |

3. **In your service workflow:**

```yaml
jobs:
  ai-report:
    uses: jinjinwang11/ai-release-monitoring/.github/workflows/deployment-report.yaml@main
    with:
      service_name: fprmfdt
      deployment_time: ${{ inputs.deployment_time }}
      commit_sha: ${{ inputs.commit_sha }}
    secrets: inherit  # ✅ Automatically inherits org secrets
```

**Benefits:**
- ✅ One-time setup for ALL services
- ✅ Centralized management
- ✅ Automatic rotation updates all services
- ✅ Matches Traveloka standard (Cursor, ECS deployment)
- ✅ No per-repo configuration needed

---

## 📋 Alternative: Repository-Level Secrets (For PoC/Testing)

If you don't have org-level access, configure per-repository:

### Setup (Per service repo)

1. **Go to Service Repo Settings**
   - https://github.com/YOUR_ORG/fpr-fprmfdt/settings/secrets/actions

2. **Create Repository Secrets:**

   | Secret Name | Value |
   |-------------|-------|
   | `DATADOG_API_KEY` | Your Datadog API key |
   | `DATADOG_APP_KEY` | Your Datadog App key |
   | `LARK_WEBHOOK` | Your Lark webhook URL |

3. **In your service workflow:**

```yaml
jobs:
  ai-report:
    uses: jinjinwang11/ai-release-monitoring/.github/workflows/deployment-report.yaml@main
    with:
      service_name: fprmfdt
      deployment_time: ${{ inputs.deployment_time }}
      commit_sha: ${{ inputs.commit_sha }}
    secrets:
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
      DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
      LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
```

---

## 🔐 Getting the Secrets

### Datadog API Keys

1. **API Key:**
   - Go to: https://app.datadoghq.com/organization-settings/api-keys
   - Click **New Key**
   - Name: `ai-monitoring` (or similar)
   - Permissions: Read-only sufficient for querying metrics
   - Copy the key

2. **Application Key:**
   - Go to: https://app.datadoghq.com/organization-settings/application-keys
   - Click **New Key**
   - Name: `ai-monitoring` (or similar)
   - Scopes: `metrics_read`, `events_read` (minimum)
   - Copy the key

**Security:**
- Keys don't auto-expire
- Recommend rotation every 6-12 months
- Can be deleted/regenerated anytime without production impact

---

### Lark Webhook

1. **Open Lark and go to your channel**
2. **Settings → Bots → Add Bot**
3. **Select "Custom Bot"**
4. **Configure:**
   - Name: `AI Deployment Reports` (or similar)
   - Description: "Automated deployment health reports"
   - Permissions: "Send messages"
5. **Copy webhook URL** (starts with `https://open.feishu.cn/open-apis/bot/v2/hook/`)

**Security:**
- Webhook can only post to that specific channel
- Can be regenerated if leaked
- Rate limited by Lark (shouldn't hit with typical deployment volume)

---

## 🎯 Recommendation for Traveloka

**Use Organization-Level Secrets** (same as Cursor agent pattern):

1. **Ask Platform/DevOps team** to create org secrets:
   - `DATADOG_API_KEY`
   - `DATADOG_APP_KEY`
   - `LARK_WEBHOOK_DEPLOYMENT_REPORTS`

2. **All services use `secrets: inherit`** in caller workflows

3. **Benefits:**
   - Consistent with existing patterns (Cursor, ECS deployment)
   - Single source of truth
   - Easy rotation
   - No per-service configuration

---

## 📝 Example: Organization-Level Setup

**Request to Platform team:**

```
Hi team,

For AI deployment monitoring PoC, could you please create these organization-level secrets?
(Same pattern as CURSOR_API_KEY for Cursor agent integration)

Secrets needed:
- DATADOG_API_KEY (I'll provide the value)
- DATADOG_APP_KEY (I'll provide the value)
- LARK_WEBHOOK_DEPLOYMENT_REPORTS (I'll provide the value)

Access: All repositories

This enables automatic deployment health reports using the same pattern
as our existing Cursor agent and ECS deployment workflows.

Thanks!
```

---

## 🔍 Reference

**Traveloka's existing patterns:**

1. **Cursor Agent** (`fpr-fprfb/.github/workflows/npe-cursor.yml`):
   ```yaml
   uses: cwang215/fpr-fprfb/.github/workflows/center-npe-cursor.yml@master
   secrets:
     CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
     CURSOR_GITHUB_TOKEN: ${{ secrets.CURSOR_GITHUB_TOKEN }}
   ```

2. **ECS Deployment** (`fpr-fprmfdt/.github/workflows/fprmfdt_ecs.yaml`):
   ```yaml
   uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml@v1
   secrets: inherit
   ```

3. **Our AI Report** (follows same pattern):
   ```yaml
   uses: jinjinwang11/ai-release-monitoring/.github/workflows/deployment-report.yaml@main
   secrets: inherit
   ```

**All three use the same reusable workflow pattern with centralized secrets!**
