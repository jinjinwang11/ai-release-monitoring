# Repo Structure

```
ai-release-monitoring/
├── .github/
│   └── workflows/
│       └── deployment-report.yaml        # 🎯 Reusable workflow (363 lines)
│                                         # Core AI deployment report logic
│
├── examples/
│   ├── standalone-workflow.yaml          # 📋 Standalone PoC (231 lines)
│   │                                     # Copy to your service for testing
│   └── caller-workflow.yaml              # 🔗 Caller example (27 lines)
│                                         # How to call reusable workflow
│
├── docs/
│   ├── POC-TEST-GUIDE.md                 # 🧪 Step-by-step PoC guide (272 lines)
│   ├── 2026-05-12-release-monitoring-platform-native.md  # 📖 Full design (1247 lines)
│   ├── 2026-05-13-INTELLIGENT-MONITORING-STRATEGY.md     # 🤖 Smart monitoring (192 lines)
│   └── 2026-05-13-REUSABLE-WORKFLOW-PATTERN.md           # 🔄 Integration pattern (99 lines)
│
├── README.md                             # 📚 Main documentation (315 lines)
└── .gitignore                            # 🚫 Git ignore rules

Total: 9 files, 1,181 lines of workflows and docs
```

---

## 🚀 Quick Start

### Option 1: Test Standalone (Recommended for PoC)

```bash
# 1. Copy to your service repo
cp examples/standalone-workflow.yaml YOUR_SERVICE/.github/workflows/deployment-report-poc.yaml

# 2. Edit file: Replace YOUR_SERVICE_NAME with actual service name

# 3. Configure GitHub secrets (see docs/POC-TEST-GUIDE.md)

# 4. Test after next production deployment
```

### Option 2: Use as Reusable Workflow

```bash
# 1. Push this repo to GitHub
git remote add origin https://github.com/YOUR_ORG/ai-release-monitoring.git
git push -u origin main

# 2. In your service repo, create caller workflow
# See: examples/caller-workflow.yaml
```

---

## 📊 What's Included

### Reusable Workflow (deployment-report.yaml)

**Core features:**
- ⏰ Wait for metrics (configurable, default 15 min)
- 📊 Query Datadog metrics (error rate, latency, memory)
- 🤖 AI analysis with GitHub Models API (GPT-4o, FREE)
- 📤 Post rich report to Lark
- 📦 Upload artifacts (report + metrics JSON)

**Inputs:**
- `service_name` - Service to monitor
- `deployment_time` - ISO 8601 timestamp
- `commit_sha` - Deployed commit
- `environment` - production/staging
- `wait_minutes` - Metrics stabilization time

**Secrets:**
- `DATADOG_API_KEY`
- `DATADOG_APP_KEY`
- `LARK_WEBHOOK`

---

### Standalone Workflow (examples/standalone-workflow.yaml)

Same functionality, but self-contained:
- ✅ No dependency on shared repos
- ✅ Good for PoC validation
- ✅ Copy directly to your service
- 🔧 Needs manual service name replacement

---

### Documentation

| Doc | Purpose | Lines |
|-----|---------|-------|
| `README.md` | Architecture, quick start, examples | 315 |
| `POC-TEST-GUIDE.md` | Step-by-step testing guide | 272 |
| `2026-05-12-release-monitoring-platform-native.md` | Full design document | 1247 |
| `2026-05-13-INTELLIGENT-MONITORING-STRATEGY.md` | Smart monitoring strategy | 192 |
| `2026-05-13-REUSABLE-WORKFLOW-PATTERN.md` | Integration pattern | 99 |

---

## 🎯 PoC Success Path

1. ✅ **Setup** (10 min)
   - Copy standalone workflow
   - Configure secrets
   - Commit to service repo

2. ⏳ **Wait** for next production deployment

3. ▶️ **Trigger** manually after deployment succeeds
   - Enter deployment time & commit SHA
   - Wait ~16 minutes

4. 🎉 **Validate** report in Lark
   - Metrics correct?
   - Analysis helpful?
   - Faster than manual monitoring?

5. 🔄 **Iterate**
   - Test 2-3 more deployments
   - Adjust metrics if needed
   - Plan rollout to other services

---

## 📈 Cost

**$0/month** - 100% free tier:
- GitHub Models API: Free (rate limited)
- GitHub Actions: Free tier (2000 min/month)
- Datadog API: Included in subscription
- Lark webhooks: Free

**Estimated usage:**
- 80 deployments/month × 16 min = 1,280 min (within free tier)

---

## 🔒 Safety

**Zero blast radius:**
- Runs AFTER deployment completes
- Separate workflow file
- Read-only operations
- Independent execution
- Worst case: No report (production unaffected)

---

## 🚦 Next Steps

After successful PoC:
1. Add more metrics (throughput, DB, cache)
2. Implement intelligent monitoring (commit-based)
3. Auto-trigger after deployment
4. Integrate with bei-backend-ci-cd-platform
5. Rollout to all services

---

## 📞 Support

- Main README: `README.md`
- Test guide: `docs/POC-TEST-GUIDE.md`
- Full design: `docs/2026-05-12-release-monitoring-platform-native.md`

---

**Built for Traveloka SRE ❤️**
