# How to Check Organization Secrets

## 🔍 Methods to Check Org-Level Secrets

---

## Method 1: GitHub Web UI (Recommended)

### Step 1: Navigate to Organization Settings

**URL:**
```
https://github.com/organizations/traveloka/settings/secrets/actions
```

**Or manually:**
1. Go to: https://github.com/traveloka
2. Click **Settings** (top right)
3. Left sidebar → **Secrets and variables** → **Actions**

### What You'll See:

**If you have access (org admin/member):**
```
Organization secrets for Actions

Name                              Updated        Repositories
────────────────────────────────────────────────────────────
AWS_ACCESS_KEY_ID                 2 days ago     All repositories
AWS_SECRET_ACCESS_KEY             2 days ago     All repositories
GITHUB_TOKEN_WORKFLOW             1 week ago     All repositories
...
```

**If you DON'T have access:**
```
403 - You need admin access to view this page
```

---

## Method 2: GitHub CLI (Requires Admin)

```bash
# List org secrets
gh api orgs/traveloka/actions/secrets

# If you get 403, you need admin permission
```

**Error you might see:**
```
403: You must be an org admin or have the actions secrets fine-grained permission.
```

---

## Method 3: Check Repository Settings

Even without org admin, you can see which secrets are **available** to a repo:

### Step 1: Go to Repository Settings

```
https://github.com/traveloka/fpr-fprmfdt/settings/secrets/actions
```

### What You'll See:

**Repository secrets section:**
- Lists repo-level secrets (if any)
- You can create/edit these if you're a repo admin

**Organization secrets section (below):**
- Lists org-level secrets **available to this repo**
- You can see names (but not values)
- Shows which ones this repo can access

**Example:**
```
Organization secrets
These secrets are available to this repository because they are set 
at the organization level.

Name                              Last updated
────────────────────────────────────────────
AWS_ACCESS_KEY_ID                 2 days ago
AWS_SECRET_ACCESS_KEY             2 days ago
DATADOG_API_KEY                   1 week ago
```

**Note:** Even without org admin, you can see secret **names** if they're available to the repo!

---

## Method 4: Infer from Workflow Runs

Check if workflows succeed without repo-level secrets:

```bash
# Check recent workflow run
gh run list --repo traveloka/fpr-fprmfdt --workflow fprmfdt_ecs.yaml --limit 5

# View a specific run
gh run view <run-id> --repo traveloka/fpr-fprmfdt
```

**Logic:**
- If deployment workflows succeed with `secrets: inherit`
- AND no secrets configured at repo level
- THEN secrets must be at org level

---

## 🎯 Quick Check Steps

### Option A: Browser (Easiest)

1. **Open in browser:**
   ```
   https://github.com/traveloka/fpr-fprmfdt/settings/secrets/actions
   ```

2. **Scroll down to "Organization secrets" section**

3. **You'll see:**
   - Secret names (not values)
   - Which ones are available to this repo
   - When they were last updated

### Option B: Ask Platform Team

If you need to know what exists:

```
Hi Platform team,

I'm working on AI deployment monitoring PoC and want to understand 
our current org-level secrets setup.

Could you help me check if these secrets already exist at org level:
- DATADOG_API_KEY
- DATADOG_APP_KEY
- LARK_WEBHOOK (or similar)

I want to avoid duplicating secrets and follow Traveloka standards.

Thanks!
```

---

## 📋 Common Traveloka Org Secrets (Likely to Exist)

Based on workflow analysis, these probably exist:

| Secret Name | Purpose | Evidence |
|-------------|---------|----------|
| `AWS_ACCESS_KEY_ID` | AWS deployment | ECS workflows succeed |
| `AWS_SECRET_ACCESS_KEY` | AWS deployment | ECS workflows succeed |
| `GITHUB_TOKEN_*` | GitHub operations | CI/CD workflows |
| `ARTIFACTORY_*` | Dependency management | Library releases |
| Various AWS account configs | Multi-account deployment | Dev/staging/prod work |

**For monitoring/observability:**
- Might exist: `DATADOG_*` (if already used)
- Might NOT exist: Lark webhooks (less common)

---

## ✅ What to Do Next

### If You Find Existing Secrets:

1. **DATADOG_API_KEY exists:**
   - ✅ Perfect! Use it directly
   - No need to create new one

2. **DATADOG_API_KEY doesn't exist:**
   - Request Platform team to create
   - Provide the key value

3. **LARK_WEBHOOK doesn't exist:**
   - This is likely (Lark is Traveloka-specific)
   - Request Platform team to create
   - Provide webhook URL

### If You Can't Access:

**Use repo-level for PoC:**
1. Configure in `ai-release-monitoring` repo
2. Test self-test workflow
3. After validation, request org-level setup

---

## 🔐 Security Note

**What you CAN see (even without admin):**
- ✅ Secret names
- ✅ When they were updated
- ✅ Which repos can access them

**What you CANNOT see:**
- ❌ Secret values (always masked)
- ❌ Who created them
- ❌ Full audit log (admin only)

This is by design - secrets are secure even for org members!

---

## 📞 Need Help?

**Can't access org settings?**
→ Check repo settings instead (you'll see available org secrets)

**Need to create new org secrets?**
→ Ask Platform/DevOps team (they have admin access)

**Want to test without org secrets?**
→ Use repo-level secrets in ai-release-monitoring for PoC

---

## 🚀 Recommended Path

**For PoC:**
1. Check `fpr-fprmfdt/settings/secrets/actions`
2. See what org secrets are available
3. If DATADOG/LARK exist → use them
4. If not → use repo-level for testing

**For Production:**
1. Request Platform team for org-level setup
2. Follow Traveloka standard (same as CI/deployment)
3. All services inherit automatically
