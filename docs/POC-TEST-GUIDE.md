# PoC Test Guide

## 🎯 Goal

Validate the AI deployment report with a real production deployment.

---

## ✅ Prerequisites

1. **Access to a service repo** (e.g., `fpr-fprmfdt`)
2. **Datadog API keys** (read-only is sufficient)
3. **Lark webhook** for notifications
4. **Upcoming production deployment** to test against

---

## 📋 Setup Steps

### Step 1: Copy Standalone Workflow (5 minutes)

```bash
# Navigate to your service repo
cd ~/Develop/fpr-fprmfdt

# Copy the standalone workflow
cp ~/Develop/ai-release-monitoring/examples/standalone-workflow.yaml \
   .github/workflows/deployment-report-poc.yaml

# Edit the file to replace YOUR_SERVICE_NAME with your actual service name
# e.g., fprmfdt, fprmfb, etc.
```

**Edit these lines in the workflow:**
- Line 34: `"title": "Deployment: YOUR_SERVICE_NAME PoC"` → `"Deployment: fprmfdt PoC"`
- Line 36: `"service:YOUR_SERVICE_NAME"` → `"service:fprmfdt"`
- Line 61: `SERVICE_NAME="YOUR_SERVICE_NAME"` → `SERVICE_NAME="fprmfdt"`
- Line 122: `SERVICE_NAME="YOUR_SERVICE_NAME"` → `SERVICE_NAME="fprmfdt"`

---

### Step 2: Configure Secrets (5 minutes)

Go to your service repo: **Settings → Secrets and variables → Actions → New repository secret**

#### A. Create `DATADOG_API_KEY`

1. Go to Datadog: https://app.datadoghq.com/organization-settings/api-keys
2. Click **New Key**
3. Name: `ai-monitoring-poc`
4. Copy the key
5. In GitHub, create secret: `DATADOG_API_KEY` = `<paste key>`

#### B. Create `DATADOG_APP_KEY`

1. Go to Datadog: https://app.datadoghq.com/organization-settings/application-keys
2. Click **New Key**
3. Name: `ai-monitoring-poc`
4. Copy the key
5. In GitHub, create secret: `DATADOG_APP_KEY` = `<paste key>`

#### C. Create `LARK_WEBHOOK`

1. Open Lark
2. Go to the channel where you want reports
3. Settings → Bots → Add Bot → Custom Bot
4. Name: `AI Deployment Reports (PoC)`
5. Copy webhook URL
6. In GitHub, create secret: `LARK_WEBHOOK` = `<paste URL>`

---

### Step 3: Commit and Push (2 minutes)

```bash
cd ~/Develop/fpr-fprmfdt

git add .github/workflows/deployment-report-poc.yaml
git commit -m "Add AI deployment report PoC workflow"
git push origin main
```

---

## 🧪 Testing

### Wait for Next Production Deployment

Monitor your team's deployment schedule. When a production deployment happens:

#### 1. Note Deployment Details

After deployment **completes successfully**:

- **Deployment time**: Check GitHub Actions timestamp (convert to ISO 8601)
  - Example: If deployed at `2026-05-14 10:32:15 +0800`
  - ISO format: `2026-05-14T02:32:15Z` (UTC)
- **Commit SHA**: Check GitHub Actions or git log
  - Example: `abc123def456...`

#### 2. Trigger PoC Workflow

1. Go to: `https://github.com/traveloka/fpr-fprmfdt/actions/workflows/deployment-report-poc.yaml`
2. Click **Run workflow** (top right)
3. Fill in:
   - **deployment_time**: `2026-05-14T02:32:15Z`
   - **commit_sha**: `abc123def456`
   - **wait_minutes**: `15` (default)
4. Click **Run workflow** (green button)

#### 3. Monitor Progress

The workflow will:
1. ⏰ Sleep 15 minutes (for metrics to stabilize)
2. 📊 Query Datadog metrics
3. 🤖 Generate AI report
4. 📤 Post to Lark

**Total time:** ~16 minutes

#### 4. Check Results

**A. GitHub Actions Log:**
- Go to: Actions → AI Deployment Report (PoC) → Latest run
- Check each step completed successfully
- Look for "✅ Report posted to Lark"

**B. Lark:**
- Check your configured channel
- You should see a blue card with:
  - 📊 Deployment Report: [service_name]
  - Metrics table
  - AI analysis
  - Recommendations

**C. Artifacts:**
- Download the artifact from GitHub Actions
- Contains: `report.md` and metric JSON files

---

## 🎯 Validation Checklist

After PoC run, validate:

- [ ] Workflow completed without errors
- [ ] Datadog event marker visible (check Datadog Events)
- [ ] Metrics queried correctly (check artifact JSON files)
- [ ] GitHub Models API call succeeded (no rate limit errors)
- [ ] AI report generated (check `report.md` in artifact)
- [ ] Report posted to Lark
- [ ] Report content makes sense
  - [ ] Metrics parsed correctly
  - [ ] Analysis is relevant
  - [ ] Recommendations are actionable

---

## 🐛 Troubleshooting

### Issue: "Error: AI call failed"

**Cause:** GitHub Models API authentication issue

**Fix:**
1. Check workflow has `permissions: models: read`
2. Verify GitHub Actions is enabled in repo settings
3. Try triggering again (may be rate limit)

---

### Issue: Empty Datadog metrics

**Cause:** Incorrect metric names for your service

**Fix:**
1. Go to Datadog → Metrics Explorer
2. Search for your service metrics
3. Update metric queries in workflow:
   - `trace.servlet.request.errors` → your actual error metric
   - `trace.servlet.request.duration` → your actual latency metric

Common alternatives:
- `trace.http.request.errors`
- `trace.api.request.errors`
- `http.server.requests.errors`

---

### Issue: Lark webhook fails

**Cause:** Webhook URL incorrect or bot disabled

**Fix:**
1. Check webhook URL is correct (starts with `https://open.feishu.cn/open-apis/bot/v2/hook/`)
2. Verify bot is still enabled in Lark channel settings
3. Test webhook manually:
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"msg_type": "text", "content": {"text": "Test message"}}'
```

---

## 📊 Example Output

**Expected Lark Card:**

```
┌─────────────────────────────────────┐
│ ✅ Healthy: fprmfdt Deployment      │ (Blue header)
├─────────────────────────────────────┤
│ ## 🚀 Deployment Report: fprmfdt   │
│                                      │
│ **Status:** ✅ Healthy              │
│ **Environment:** production          │
│ **Deployment Time:** 2026-05-14...  │
│ **Commit:** abc123                   │
│                                      │
│ ### 📊 Metrics                      │
│ | Metric | Before | After | ...     │
│ | Error Rate | 0.12% | 0.11% | ... │
│ | Latency | 245ms | 238ms | ...     │
│                                      │
│ ### 🔍 Analysis                     │
│ Deployment successful...             │
│                                      │
│ ### 💡 Recommendation               │
│ ✅ No action needed.                │
├─────────────────────────────────────┤
│ 🤖 Generated by AI Release          │
│    Monitoring (PoC)                  │
└─────────────────────────────────────┘
```

---

## 📈 Success Criteria

PoC is successful if:

1. ✅ Workflow runs without errors
2. ✅ Datadog metrics queried correctly
3. ✅ AI generates meaningful analysis
4. ✅ Report arrives in Lark within 20 minutes
5. ✅ Report helps understand deployment health
6. ✅ Faster than manual monitoring (30-60 min → 0 min hands-on)

---

## 🚀 Next Steps After Success

1. **Run 2-3 more tests** with different deployments
2. **Gather feedback** from team
3. **Iterate on metrics** if needed
4. **Add more metrics** (memory, throughput, etc.)
5. **Consider automation** (auto-trigger after deployment)
6. **Plan rollout** to other services

---

## 📞 Need Help?

- Check main README: `/Users/jinjin.wang/Develop/ai-release-monitoring/README.md`
- Review design doc: `/Users/jinjin.wang/docs/superpowers/specs/2026-05-12-release-monitoring-platform-native.md`
- Check Datadog API docs: https://docs.datadoghq.com/api/
- GitHub Models API: https://docs.github.com/en/github-models

---

**Good luck with your PoC! 🚀**
