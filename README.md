# AI Release Monitoring (PoC)

🤖 **AI-powered deployment monitoring for Traveloka microservices**

## 🎯 Purpose

Automate post-deployment monitoring with AI-generated health reports:
- ✅ Query Datadog metrics before/after deployment
- ✅ Analyze commits to understand code changes
- ✅ Use GitHub Models API (free!) for intelligent analysis
- ✅ Post detailed reports to Lark
- ✅ Identify suspect commits for anomalies

**Replaces:** 30-60 minutes of manual Datadog monitoring per deployment

**Cost:** $0/month (100% free tier - GitHub Models API + Datadog API)

---

## 🏗️ Architecture

```
┌──────────────────────┐
│ Production Deploy    │ ← Engineer approves & deploys
│   (Manual Gate)      │
└──────────┬───────────┘
           │ ✅ Deploy succeeds
           │
           ▼
┌──────────────────────┐
│ Wait 15 min          │ ← Let metrics stabilize
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Query Datadog        │ ← Get metrics before/after
│  - Error rate        │
│  - Latency P95       │
│  - Memory usage      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ GitHub Models API    │ ← Free AI analysis (GPT-4o)
│  Analyze:            │
│  - Metrics changes   │
│  - Commit diffs      │
│  - Potential issues  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Post to Lark         │ ← Send rich report card
└──────────────────────┘
```

---

## 📦 What's Included

### Reusable Workflow (Recommended)

**File:** `.github/workflows/deployment-report.yaml`

- ✅ Can be reused by all services (like Cursor agent pattern)
- ✅ Centralized updates and improvements
- ✅ Consistent across all services

**Usage in your service:**

```yaml
# In fpr-fprmfdt/.github/workflows/call-ai-report.yaml
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

### Standalone Workflow (For Testing)

**File:** `examples/standalone-workflow.yaml`

- ✅ Copy directly to your service for quick testing
- ✅ No dependencies on shared repos
- ✅ Good for PoC validation

---

## 🚀 Quick Start (PoC)

### Option 1: Test in Your Service (Recommended for PoC)

**Step 1: Copy standalone workflow**

```bash
cp examples/standalone-workflow.yaml YOUR_SERVICE/.github/workflows/deployment-report-poc.yaml
```

**Step 2: Configure secrets in your repo**

GitHub repo → Settings → Secrets and variables → Actions:

| Secret | Where to Get |
|--------|-------------|
| `DATADOG_API_KEY` | Datadog → Organization Settings → API Keys → Create Key |
| `DATADOG_APP_KEY` | Datadog → Organization Settings → Application Keys → Create Key |
| `LARK_WEBHOOK` | Lark → Create bot → Copy webhook URL |

**Step 3: Test after next production deployment**

1. ⏳ Wait for production deployment to complete successfully
2. 📝 Note deployment time and commit SHA from GitHub Actions
3. ▶️ Manually trigger PoC workflow:
   - Go to: Actions → "AI Deployment Report (PoC)" → Run workflow
   - Enter deployment time (ISO 8601, e.g., `2026-05-14T10:30:00Z`)
   - Enter commit SHA
4. ⏰ Wait ~16 minutes (15min sleep + 1min execution)
5. ✅ Check Lark for the AI-generated report!

---

### Option 2: Use as Reusable Workflow

**Step 1: Push this repo to GitHub**

```bash
git remote add origin https://github.com/jinjinwang11/ai-release-monitoring.git
git push -u origin main
```

**Step 2: Create caller workflow in your service**

See `examples/caller-workflow.yaml` for template.

---

## 📊 Example Report

```markdown
## Deployment Report: fprmfdt

**Status:** ✅ Healthy
**Deployment Time:** 2026-05-14T10:32:00Z
**Commit:** abc123

### Metrics

| Metric | Before | After | Change | Status |
|--------|--------|-------|--------|--------|
| Error Rate | 0.12% | 0.11% | -8.3% | ✅ Improved |
| Latency P95 | 245ms | 238ms | -2.9% | ✅ Stable |
| Memory Usage | 68% | 70% | +2.9% | ✅ Normal |

### Analysis

Deployment successful. Error rate slightly improved, likely due to fix in 
payment validation (commit abc123). Memory usage increased minimally, 
within expected range for this service.

### Recent Commits

- abc123: Fix payment validation edge case
- def456: Update Redis client timeout
- ghi789: Refactor logging

### Recommendation

✅ **No action needed.** All metrics healthy. Monitor memory usage over 
next 24h to ensure no gradual leak.
```

---

## ⚙️ How It Works

### 1. Wait for Metrics (15 minutes)

After deployment, metrics need time to stabilize. The workflow sleeps 15 minutes.

### 2. Query Datadog

Uses Datadog API to query metrics in two windows:
- **Before:** 1 hour before deployment
- **After:** 15 minutes after deployment

Example query:
```bash
curl -G "https://api.datadoghq.com/api/v1/query" \
  -H "DD-API-KEY: $DATADOG_API_KEY" \
  -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
  --data-urlencode "query=avg:trace.servlet.request.errors{service:fprmfdt,env:production}" \
  -d "from=$BEFORE_START" \
  -d "to=$BEFORE_END"
```

### 3. AI Analysis

Sends to GitHub Models API (GPT-4o):
- Metrics before/after
- Recent commits
- Service context

**Cost:** FREE (rate limits: 10 req/min, 50 req/day)

### 4. Post to Lark

Formats as rich Lark card and sends via webhook.

---

## 🔒 Safety Guarantees

**Zero blast radius for production!**

| Layer | Protection |
|-------|-----------|
| **Time** | Runs AFTER deployment completes (not during) |
| **File** | Separate workflow file (doesn't touch deployment workflow) |
| **Execution** | Independent job (no dependencies on deployment) |
| **Operation** | Read-only (only queries, no writes to production) |
| **Failure** | If PoC fails, production already succeeded |

**Worst case:** PoC workflow fails → No report sent → No production impact

---

## 📈 Roadmap

### ✅ Phase 1: PoC (Current)
- Manual trigger
- Baseline metrics (error, latency, memory)
- Basic AI analysis
- Lark notification

### 🔄 Phase 2: Intelligent Monitoring
- AI analyzes commits to select dynamic metrics
  - Redis changes → monitor redis metrics
  - DB changes → monitor DB metrics
- Smart anomaly detection based on code changes

### 🎯 Phase 3: Full Integration
- Auto-trigger after successful deployment
- Integrate with `bei-backend-ci-cd-platform`
- Delayed monitoring (1h, 4h, 24h checkpoints)
- Rollout to all services

### 🚀 Phase 4: Advanced Features
- Datadog Monitors as backup layer
- Suspect commit identification (binary search)
- Daily release notes (auto-update Lark docs)
- Trend analysis across deployments

---

## 🛠️ Customization

### Add More Metrics

Edit the "Query Datadog Metrics" step:

```yaml
- name: Query Datadog Metrics
  run: |
    # Add latency query
    BEFORE_LATENCY=$(curl -G "https://api.datadoghq.com/api/v1/query" \
      -H "DD-API-KEY: $DATADOG_API_KEY" \
      -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
      --data-urlencode "query=avg:trace.servlet.request.duration{service:$SERVICE_NAME}" \
      -d "from=$BEFORE_START" -d "to=$BEFORE_END")
```

### Adjust Wait Time

Change sleep duration:

```yaml
- name: Wait for Metrics
  run: sleep 300  # 5 minutes instead of 15
```

### Customize AI Prompt

Edit the prompt in "Generate AI Report" step to focus on specific aspects.

---

## 📚 Documentation

- [Full Design Doc](docs/2026-05-12-release-monitoring-platform-native.md)
- [Intelligent Monitoring Strategy](docs/2026-05-13-INTELLIGENT-MONITORING-STRATEGY.md)
- [Safety Guarantees (中文)](docs/2026-05-13-PoC-SAFETY-GUARANTEES.md)
- [Reusable Workflow Pattern](docs/2026-05-13-REUSABLE-WORKFLOW-PATTERN.md)

---

## 🤝 Contributing

This is a PoC. Feedback and improvements welcome!

---

## 📞 Support

Questions? Check the design docs or reach out to the team.

---

**Built with ❤️ for Traveloka SRE**
