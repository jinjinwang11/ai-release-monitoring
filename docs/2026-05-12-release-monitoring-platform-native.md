# AI-Powered Release Monitoring with GitHub Models API

**Date:** 2026-05-13  
**Status:** Ready for PoC  
**Author:** AI Assistant

**Version History:**
- v1 (2026-05-12): Initial platform-native design with Datadog Monitors
- v2 (2026-05-13): Updated with GitHub Models API, analyzed actual deployment flow, added PoC plan

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement & Requirements](#2-problem-statement--requirements)
3. [Solution Architecture](#3-solution-architecture)
4. [AI-Powered Deployment Report (Detailed Implementation)](#4-ai-powered-deployment-report-detailed-implementation)
5. [Complete Deployment Workflow with AI Report](#5-complete-deployment-workflow-with-ai-report)
6. [Daily Release Notes](#6-daily-release-notes)
7. [Implementation Steps (Outdated)](#7-implementation-steps-outdated---replaced-by-section-11)
8. [Costs & Resources](#8-costs--resources)
9. [Comparison: AI-Powered vs Alternatives](#9-comparison-ai-powered-vs-alternatives)
10. [Key Capabilities](#10-key-capabilities)
11. [Implementation Steps](#11-implementation-steps)
12. [PoC (Proof of Concept) - Executable Plan](#12-poc-proof-of-concept---executable-plan) ⭐ **START HERE**
- [Appendix A: Example AI Report Output](#appendix-a-example-ai-report-output)
- [Appendix B: Datadog API Authentication](#appendix-b-datadog-api-authentication)

## 1. Executive Summary

This document describes a **fully automated, AI-powered** post-deployment monitoring solution using **100% platform-native** features:
- **GitHub Models API** (free AI intelligence)
- **Datadog API** (metrics and monitoring)
- **GitHub Actions** (serverless compute)
- **Lark webhooks** (notifications)

**Key Principle:** Use free platform capabilities to achieve intelligent monitoring with ZERO external costs and ZERO custom infrastructure.

### What Changed From Previous Design

**Previous approach (v1):**
- Custom GitHub Actions workflows running Python scripts
- Custom AI-powered anomaly detection logic
- Required external AI APIs (Anthropic/OpenAI) at $12-50/month
- Complex delayed monitoring with state management

**Platform-native approach (v2 - THIS DOCUMENT):**
- GitHub Models API provides free AI intelligence (GPT-4o, Claude, Llama)
- Datadog API for metric queries and deployment tracking
- AI adapts to each deployment (analyzes commits + metrics)
- Intelligent suspect commit identification
- 100% runs in GitHub Actions (no servers)

### Benefits

✅ **100% Free:** GitHub Models API rate-limited free tier (50 requests/day)  
✅ **Intelligent:** AI analyzes metrics with context, not just thresholds  
✅ **Adaptive:** AI selects relevant metrics based on code changes  
✅ **No infrastructure:** Runs entirely in GitHub Actions  
✅ **No maintenance:** No servers, no external API keys to rotate  
✅ **Automated:** Reports generated 15 minutes after each deployment

---

## 2. Problem Statement & Requirements

### Current Pain Points

**Manual Post-Deployment Monitoring:**
- Engineers manually check Datadog after each production deployment
- 30-60 minutes spent per deployment on verification
- No systematic process - depends on engineer's experience
- Easy to miss subtle issues or slow-growing problems

**Lack of Anomaly Detection:**
- No automated comparison of pre/post-deployment metrics
- Hard to distinguish normal variance from real issues
- Subtle degradations go unnoticed until customer complaints

**Difficult Root Cause Analysis:**
- When issues found, manual git bisect through commits
- Time-consuming to correlate anomalies with specific code changes
- No automatic suspect identification

**No Release Visibility:**
- No centralized daily summary of what was deployed
- Hard to track deployment health across multiple services
- Manual effort to generate release notes

### Core Requirements

**R1: Automated Post-Deployment Monitoring**
- Automatically monitor production after each deployment
- Compare metrics before/after deployment
- Detect anomalies without human intervention
- Target: Alert within 5-15 minutes for immediate issues

**R2: Two-Layer Anomaly Detection**

**Layer 1 - Immediate Issues (0-15 minutes):**
- Detect sudden spikes: error rates, latency, throughput drops
- Hard threshold-based: e.g., error rate >50% increase = alert
- Examples: API 500 errors, latency jumps, throughput drops

**Layer 2 - Slow-Growing Issues (1-24 hours):**
- Detect gradual accumulation: Redis memory growth, storage creep
- Trend-based detection: "Is this normal variance or concerning?"
- Example: Redis memory 1GB → 1.2GB → 1.5GB over hours

**R3: Lark Integration**
- Post monitoring results to Lark channel
- Alert format: ✅ Success / ⚠️ Warning / 🔴 Critical
- Include metrics, Datadog links, recommendations

**R4: Daily Release Notes**
- Automated daily summary (6pm SGT, Mon-Fri)
- Aggregate all deployments from last 24 hours
- Update to Lark Doc

### Success Criteria

- Reduce manual monitoring time: 30-60 mins → <5 mins per deployment
- Immediate anomaly detection: <15 minutes
- Slow-growth detection: <4 hours
- False positive rate: <20%
- 100% of production deployments monitored

---

## 3. Solution Architecture

### High-Level Flow

```
1. Engineer creates deployment (pushes to main/release/* branch)
   ↓
2. GitHub Actions workflow starts:
   - Builds service
   - Deploys to STAGING automatically
   ↓
3. Engineer sends approval request message with GitHub Actions link
   - Other engineers review and approve in GitHub Actions UI
   ↓
4. Production deployment starts (after approval):
   - Deploys to PRODUCTION ECS
   - Marks deployment in Datadog (Events API)
   - Creates GitHub release
   ↓
5. Wait 15 minutes for metrics to stabilize
   ↓
6. AI Report Generation (GitHub Actions):
   - Queries Datadog metrics (before/after deployment)
   - Fetches commit changes (GitHub API)
   - Calls GitHub Models API (GPT-4o) with context:
     * What changed (commits)
     * Metric values before/after
     * Service context
   - AI analyzes and generates intelligent report
   ↓
7. Post Report to Lark:
   - Deployment summary with status (✅/⚠️/❌)
   - Metrics comparison table
   - Anomaly analysis (if any)
   - Suspect commits (if issues detected)
   - Recommendations (rollback/monitor/all clear)
```

### Real Deployment Flow (Based on bei-backend-ci-cd-platform)

**Current Process:**
1. Engineer pushes to `main`, `release/**`, or `hotfix/**` branch
2. GitHub Actions workflow (`fprmfdt_ecs.yaml`) triggers
3. Workflow uses reusable `traveloka/bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml@v1`
4. **STAGING deployment** happens automatically
5. **PRODUCTION deployment** requires manual approval:
   - Engineer sends GitHub Actions link (e.g., https://github.com/traveloka/fpr-fprinv/actions/runs/25774769625/job/75706749050)
   - Another engineer approves in GitHub Actions UI
   - Deployment proceeds to PRODUCTION
6. **After deployment succeeds:**
   - ✅ Currently: Manual monitoring (30-60 min)
   - 🎯 **New: Automated AI report (15 min wait + analysis)**

### Key Components

**Component 1: Extend Existing Deployment Workflow**
- Add post-deployment steps to `bei-backend-ci-cd-platform/ecs_deployment.yaml`
- OR: Modify each service's `fprmfdt_ecs.yaml` to call a separate report workflow
- Triggers ONLY after PRODUCTION deployment succeeds
- **Zero changes to approval flow** - AI report happens AFTER approval + deployment

**Component 2: AI-Powered Deployment Report**
- Runs automatically after production deployment completes
- GitHub Actions workflow with:
  - 15-minute wait for metrics stabilization
  - Datadog API queries (metrics before/after)
  - GitHub Models API call (AI analysis)
  - Lark webhook (post report)
- **Zero external costs:** All free platform APIs

**Component 3: GitHub Models API Integration**
- Free AI models (GPT-4o, Claude, Llama-3.3-70B)
- Rate limits: 10-15 req/min, 50-150 req/day
- Authentication: Built-in `${{ secrets.GITHUB_TOKEN }}`
- No API key management required

**Component 4: Datadog Deployment Tracking**
- Mark deployment in Datadog Events API
- Enables before/after metric comparison
- Links deployment to metric anomalies

**Component 5: Optional Backup - Datadog Monitors**
- Anomaly Detection monitors as safety net
- Alert on critical issues independently of AI
- Provides redundancy layer

**Component 6: Daily Summary Report**
- Single GitHub Action scheduled workflow
- Runs 6pm SGT, Mon-Fri
- Aggregates deployment reports from Lark
- Posts summary to Lark Doc

---

## 4. AI-Powered Deployment Report (Detailed Implementation)

### 4.1 GitHub Models API Setup

**What:** GitHub's free AI API that provides access to multiple models (GPT-4o, Claude, Llama, etc.) without external API keys.

**Available Models:**
- `gpt-4o` - 10 req/min, 50 req/day (default)
- `claude-3.5-sonnet` - 10 req/min, 50 req/day
- `llama-3.3-70b` - 15 req/min, 150 req/day

**Authentication:** Use built-in `${{ secrets.GITHUB_TOKEN }}` (no external API keys!)

**Rate Limits:**
- 20 deployments/week = ~3/day
- Well within 50 requests/day limit ✅

**Endpoint:** `https://models.inference.ai.azure.com/chat/completions`

**Required Permission:** Add to workflow:
```yaml
permissions:
  models: read  # Access GitHub Models API
```

### 4.2 Deployment Tracking in Datadog

**What:** Mark each deployment as an event so metrics can be queried with proper time windows.

**How:** Add to deployment workflow (GitHub Actions):

```bash
# After successful ECS deployment
DEPLOYMENT_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

curl -X POST "https://api.datadoghq.com/api/v1/events" \
  -H "DD-API-KEY: ${DATADOG_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Deployment: booking-api v1.2.3",
    "text": "Deployed booking-api v1.2.3 to production",
    "tags": [
      "deployment",
      "service:booking-api",
      "env:production",
      "version:1.2.3"
    ]
  }'
```

**Result:** Deployment timestamp recorded for before/after metric comparison.

### 4.3 AI Report Generation Workflow

**How AI Works:**

1. **Input Context:**
   - Service name, version, deployment time
   - Commit changes (what code changed)
   - Datadog metrics before/after deployment
   
2. **AI Analysis:**
   - Parses metric values from Datadog API responses
   - Compares before vs after (% change, absolute values)
   - Identifies anomalies (>20% latency, >50% errors = concerning)
   - Cross-references metrics with commit changes
   - Identifies suspect commits if issues detected
   
3. **Output:**
   - Deployment status (✅ Healthy / ⚠️ Warning / ❌ Critical)
   - Metrics comparison table
   - Natural language analysis
   - Suspect commit identification
   - Recommendations (rollback/monitor/all clear)

**Example AI Prompt:**

```
You are an SRE assistant analyzing a production deployment.

Service: booking-api
Version: v1.2.3
Deployment Time: 2026-05-13T14:32:00Z

Commits in this deployment:
- abc123: Add Redis caching for user sessions
- def456: Optimize database query in booking endpoint
- ghi789: Update payment validation logic

Datadog Metrics Comparison:

Error Rate (before deployment, 1h avg):
{"series": [{"pointlist": [[1715612400, 0.12]]}]}

Error Rate (after deployment, 15min avg):
{"series": [{"pointlist": [[1715615820, 0.35]]}]}

Latency P95 (before deployment, 1h avg):
{"series": [{"pointlist": [[1715612400, 245]]}]}

Latency P95 (after deployment, 15min avg):
{"series": [{"pointlist": [[1715615820, 268]]}]}

Your task:
1. Parse metric values from JSON
2. Compare before vs after (calculate % change)
3. Identify significant changes (>20% latency, >50% errors)
4. If issues found, analyze commits for likely cause
5. Generate deployment health report

Report format: [see template below]
```

**AI Output Example:**

```markdown
## Deployment Report: booking-api v1.2.3
**Status:** ⚠️ Warning - Elevated Error Rate

### Metrics Comparison
| Metric | Before | After | Change | Assessment |
|--------|--------|-------|--------|------------|
| Error Rate | 0.12% | 0.35% | +192% | ⚠️ Warning |
| Latency P95 | 245ms | 268ms | +9% | ✅ Normal |

### Analysis
Error rate increased significantly from 0.12% to 0.35% (192% increase, +0.23pp).
This correlates with deployment timing and exceeds normal variance.

Latency shows minor increase (9%) which is within acceptable range.

### Suspect Commits
**ghi789 - Update payment validation logic**
This commit modified payment validation which could cause errors if validation
logic is too strict or has edge cases not covered by tests.

### Recommendation
⚠️ Monitor closely for next 1-2 hours. If error rate continues above 0.3%,
consider rollback. Review payment validation logic for edge cases.
```

### 4.4 Datadog Monitors (Optional Backup Layer)

**Purpose:** Provide redundancy if AI report generation fails or for critical alerts.

**Setup (via Datadog UI) - OPTIONAL:**

Create basic threshold monitors for critical metrics:

1. **Critical Error Rate Monitor:**
   - Metric: `trace.servlet.request.errors{service:booking-api}`
   - Alert when: `> 1%` for 5 minutes
   - Notification: Lark webhook
   
2. **Critical Latency Monitor:**
   - Metric: `trace.servlet.request.duration.by.service.95p{service:booking-api}`
   - Alert when: `> 1000ms` for 5 minutes
   - Notification: Lark webhook

**Note:** These are safety nets. Primary intelligence comes from AI reports.

---

## 5. Complete Deployment Workflow with AI Report

### 5.1 Integration Approach

**Option A: Modify Reusable Workflow (Recommended)**
- Add AI report generation to `bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml`
- Benefits: One-time change, all services get it automatically
- Requires: Access to modify `bei-backend-ci-cd-platform` repo

**Option B: Per-Service Workflow Call**
- Each service calls a separate `deployment-report.yaml` workflow after deployment
- Benefits: No changes to shared platform, easier to test
- Requires: Adding step to each service's workflow

**We'll document Option B below** (easier to pilot and rollout):

### 5.2 Deployment Report Workflow (Shared)

**Location:** Create in a shared repo (e.g., `bei-monitoring` or `bei-backend-ci-cd-platform`)

**File:** `.github/workflows/deployment-report.yaml`

```yaml
name: AI Deployment Report

on:
  workflow_call:
    inputs:
      service_name:
        required: true
        type: string
      environment:
        required: true
        type: string
        description: 'staging or production'
      deployment_time:
        required: true
        type: string
        description: 'ISO 8601 timestamp'
      commit_sha:
        required: true
        type: string
    secrets:
      DATADOG_API_KEY:
        required: true
      DATADOG_APP_KEY:
        required: true
      LARK_WEBHOOK:
        required: true

permissions:
  contents: read
  models: read  # For GitHub Models API

jobs:
  generate-report:
    # Only run for production deployments
    if: inputs.environment == 'production'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ inputs.commit_sha }}
          fetch-depth: 20  # Fetch recent commits for analysis
      
      - name: Wait for Metrics Stabilization
        run: |
          echo "Waiting 15 minutes for metrics to accumulate..."
          sleep 900  # 15 minutes
      
      - name: Mark Deployment in Datadog
        run: |
          curl -X POST "https://api.datadoghq.com/api/v1/events" \
            -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{
              "title": "Deployment: ${{ inputs.service_name }} to ${{ inputs.environment }}",
              "text": "Deployed version ${{ inputs.commit_sha }}",
              "tags": [
                "deployment",
                "service:${{ inputs.service_name }}",
                "env:${{ inputs.environment }}",
                "commit:${{ inputs.commit_sha }}"
              ],
              "date_happened": '"$(date -d "${{ inputs.deployment_time }}" +%s)"'
            }'
      
      - name: Generate AI Deployment Report
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
          DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
          LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
          SERVICE_NAME: ${{ inputs.service_name }}
          DEPLOYMENT_TIME: ${{ inputs.deployment_time }}
          COMMIT_SHA: ${{ inputs.commit_sha }}
        run: |
          # Calculate time windows
          BEFORE_START=$(date -u -d "$DEPLOYMENT_TIME - 1 hour" +%s)
          BEFORE_END=$(date -u -d "$DEPLOYMENT_TIME" +%s)
          AFTER_START=$(date -u -d "$DEPLOYMENT_TIME" +%s)
          AFTER_END=$(date -u +%s)  # Now (15 min after deployment)
          
          # Query Datadog metrics (before deployment)
          echo "Querying metrics before deployment..."
          BEFORE_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            --fail-with-body \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:$SERVICE_NAME,env:production}" \
            -d "from=$BEFORE_START" \
            -d "to=$BEFORE_END")
          
          BEFORE_LATENCY=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            --fail-with-body \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.duration.by.service.95p{service:$SERVICE_NAME,env:production}" \
            -d "from=$BEFORE_START" \
            -d "to=$BEFORE_END")
          
          # Query Datadog metrics (after deployment)
          echo "Querying metrics after deployment..."
          AFTER_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            --fail-with-body \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:$SERVICE_NAME,env:production}" \
            -d "from=$AFTER_START" \
            -d "to=$AFTER_END")
          
          AFTER_LATENCY=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            --fail-with-body \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.duration.by.service.95p{service:$SERVICE_NAME,env:production}" \
            -d "from=$AFTER_START" \
            -d "to=$AFTER_END")
          
          # Get commits in this deployment
          COMMITS=$(git log --oneline --no-decorate -10)
          
          # Create AI prompt
          read -r -d '' AI_PROMPT << 'EOF'
          You are an SRE assistant analyzing a production deployment.
          
          Service: $SERVICE_NAME
          Commit: $COMMIT_SHA
          Deployment Time: $DEPLOYMENT_TIME
          
          Commits in this deployment:
          $COMMITS
          
          Datadog Metrics Comparison:
          
          Error Rate (before deployment, 1h avg):
          $BEFORE_ERRORS
          
          Error Rate (after deployment, 15min avg):
          $AFTER_ERRORS
          
          Latency P95 (before deployment, 1h avg):
          $BEFORE_LATENCY
          
          Latency P95 (after deployment, 15min avg):
          $AFTER_LATENCY
          
          Your task:
          1. Parse the metric values from the JSON responses (look for "pointlist" arrays)
          2. Compare before vs after metrics (calculate % change and absolute difference)
          3. Identify any significant changes:
             - Error rate increase >50% = Warning
             - Error rate increase >100% = Critical
             - Latency increase >20% = Warning
             - Latency increase >50% = Critical
          4. If issues detected, analyze commits to identify likely causes
          5. Generate a deployment health report
          
          Report format (use markdown):
          ## Deployment Report: $SERVICE_NAME
          **Deployment Time:** $DEPLOYMENT_TIME
          **Commit:** $COMMIT_SHA
          **Status:** [Choose: ✅ Healthy / ⚠️ Warning / ❌ Issues Detected]
          
          ### Metrics Comparison
          | Metric | Before | After | Change | Assessment |
          |--------|--------|-------|--------|------------|
          | Error Rate | X% | Y% | +Z% | [✅ Normal / ⚠️ Warning / ❌ Critical] |
          | Latency P95 | Xms | Yms | +Zms | [✅ Normal / ⚠️ Warning / ❌ Critical] |
          
          ### Analysis
          [Explain what changed and why it matters - 2-3 sentences]
          
          ### Suspect Commits (if issues found)
          [List commits that likely caused issues with reasoning]
          
          ### Recommendation
          [Should we: rollback immediately / monitor closely / all clear?]
          EOF
          
          # Replace variables in prompt
          AI_PROMPT=$(echo "$AI_PROMPT" | envsubst)
          
          # Call GitHub Models API (GPT-4o)
          echo "Calling GitHub Models API for analysis..."
          AI_REPORT=$(curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $GITHUB_TOKEN" \
            -d @- << JSON_END | jq -r '.choices[0].message.content // "Error: AI analysis failed"'
          {
            "model": "gpt-4o",
            "messages": [
              {
                "role": "system",
                "content": "You are an expert SRE analyzing deployment health metrics. Be concise but thorough. Focus on actionable insights."
              },
              {
                "role": "user",
                "content": $(echo "$AI_PROMPT" | jq -Rs .)
              }
            ],
            "max_tokens": 2000,
            "temperature": 0.3
          }
          JSON_END
          )
          
          # Post report to Lark
          echo "Posting report to Lark..."
          curl -X POST "$LARK_WEBHOOK" \
            -H "Content-Type: application/json" \
            -d @- << JSON_END
          {
            "msg_type": "interactive",
            "card": {
              "header": {
                "title": {
                  "content": "📊 Deployment Report: $SERVICE_NAME",
                  "tag": "plain_text"
                },
                "template": "blue"
              },
              "elements": [
                {
                  "tag": "markdown",
                  "content": $(echo "$AI_REPORT" | jq -Rs .)
                },
                {
                  "tag": "hr"
                },
                {
                  "tag": "action",
                  "actions": [
                    {
                      "tag": "button",
                      "text": {
                        "content": "View in Datadog",
                        "tag": "plain_text"
                      },
                      "url": "https://app.datadoghq.com/apm/service/$SERVICE_NAME/operations/servlet.request",
                      "type": "primary"
                    },
                    {
                      "tag": "button",
                      "text": {
                        "content": "View Commit",
                        "tag": "plain_text"
                      },
                      "url": "https://github.com/${{ github.repository }}/commit/$COMMIT_SHA",
                      "type": "default"
                    }
                  ]
                }
              ]
            }
          }
          JSON_END
          
          echo "✅ Deployment report posted to Lark"
```

### 5.3 Per-Service Integration

**Modify each service's deployment workflow** (e.g., `fpr-fprmfdt/.github/workflows/fprmfdt_ecs.yaml`)

**Option A: Modify the generated file's template** (in service's build.gradle or generator config)

OR

**Option B: Add manual step after deployment** (if generator allows custom steps):

```yaml
jobs:
  standard_deployment:
    uses: traveloka/bei-backend-ci-cd-platform/.github/workflows/ecs_deployment.yaml@v1
    with:
      # ... existing config ...
    secrets: inherit
  
  # NEW: AI deployment report
  deployment_report:
    needs: standard_deployment
    if: success()  # Only if deployment succeeded
    uses: traveloka/bei-monitoring/.github/workflows/deployment-report.yaml@v1
    with:
      service_name: fprmfdt
      environment: production  # or extract from deployment job
      deployment_time: ${{ needs.standard_deployment.outputs.deployment_time }}
      commit_sha: ${{ github.sha }}
    secrets:
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
      DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
      LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
```

**Note:** If `ecs_deployment.yaml` doesn't output `deployment_time`, we can use `${{ github.event.head_commit.timestamp }}` or capture it in the report workflow.

### 5.4 Important Notes on Deployment Flow

**Manual Approval Flow (Unchanged):**
1. Engineer creates deployment by pushing to `main`, `release/**`, or `hotfix/**`
2. GitHub Actions starts, deploys to STAGING automatically
3. Engineer sends GitHub Actions link to team (e.g., "Ready to deploy fprmfdt, please approve: https://github.com/traveloka/fpr-fprinv/actions/runs/25774769625/job/75706749050")
4. **Another engineer approves in GitHub Actions UI** (required for production)
5. Deployment to PRODUCTION proceeds
6. **AI report generation happens AFTER production deployment succeeds** (15 min wait + analysis)

**Zero Impact on Existing Flow:**
- ✅ No changes to approval process
- ✅ No changes to deployment mechanics
- ✅ AI report is **additive only** - runs after everything succeeds
- ✅ If AI report fails, deployment is still successful (reports are informational)

**Required Secrets (add at org level or per repo):**
- `DATADOG_API_KEY` - Datadog API key (for marking deployments)
- `DATADOG_APP_KEY` - Datadog Application key (for querying metrics)
- `LARK_WEBHOOK` - Lark webhook URL for deployment reports

---

## 6. Daily Release Notes

### 6.1 GitHub Action (Scheduled)

**Create:** `.github/workflows/daily-release-report.yml` in ONE repository (e.g., ops repo or any service)

```yaml
name: Daily Release Report
on:
  schedule:
    - cron: '0 10 * * 1-5'  # 6pm SGT = 10am UTC, Mon-Fri

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - name: Fetch releases from all repos
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          LARK_DOC_WEBHOOK: ${{ secrets.LARK_DOC_WEBHOOK }}
        run: |
          # List of your service repos
          REPOS=("booking-api" "payment-service" "user-service")
          
          # Fetch releases from last 24 hours
          REPORT="📋 Daily Release Report - $(date +%Y-%m-%d)\n\n"
          
          for repo in "${REPOS[@]}"; do
            releases=$(gh release list --repo traveloka/$repo --limit 10 --json tagName,publishedAt,author)
            # Filter to last 24 hours and format
            # (add date filtering logic here)
            REPORT="${REPORT}\n## $repo\n${releases}\n"
          done
          
          # Post to Lark Doc
          curl -X POST "${LARK_DOC_WEBHOOK}" \
            -H "Content-Type: application/json" \
            -d "{\"text\": \"${REPORT}\"}"
```

**Alternative (Simpler):** Instead of parsing releases, just aggregate the deployment notifications from Lark channel and post summary.

---

## 7. Implementation Steps (Outdated - Replaced by Section 11)

**Note:** This section describes the old implementation flow and is superseded by Section 11 which reflects the AI-powered approach. Kept for reference only.



## 8. Costs & Resources

### Platform Costs (100% FREE!)

| Component | Provider | Monthly Cost |
|-----------|----------|--------------|
| AI analysis (GPT-4o) | GitHub Models API | **$0** (50 req/day free tier) |
| GitHub Actions compute | GitHub | **$0** (within 2,000 min/month free tier) |
| Datadog API calls | Datadog | **$0** (included in subscription) |
| Lark webhooks | Lark | **$0** (unlimited) |
| **Total** | | **$0/month** |

### Resource Usage Estimate

**Per deployment:**
- GitHub Actions: ~18 minutes (15 min wait + 2-3 min execution)
- GitHub Models API: 1 request
- Datadog API: 4-6 queries

**Monthly (20 deployments/week × 4 weeks = 80 deployments):**
- GitHub Actions: 1,440 minutes (within 2,000 free limit ✅)
- GitHub Models API: 80 requests (within 1,500/month limit ✅)
- Well within all free tiers!

### Human Time

- **Initial setup:** 2-3 days (GitHub workflow template creation)
- **Per-service setup:** 15-30 minutes (copy workflow, add secrets)
- **Ongoing maintenance:** <1 hour/month (review reports, tune if needed)

**Total:** Minimal ongoing cost - 100% automated!

---

## 9. Comparison: AI-Powered vs Alternatives

| Aspect | AI-Powered (GitHub Models) | Datadog Monitors Only | External AI (Anthropic/OpenAI) |
|--------|---------------------------|----------------------|-------------------------------|
| **Intelligence** | ✅ AI analyzes with context | ❌ Threshold-based only | ✅ AI analyzes with context |
| **Adaptability** | ✅ Adapts to code changes | ❌ Static configuration | ✅ Adapts to code changes |
| **Monthly cost** | **$0** | **$0** | **$12-50** |
| **Setup complexity** | Low (copy workflow) | Medium (configure monitors) | Low (copy workflow + API key) |
| **Maintenance** | Minimal (platform updates) | Medium (tune thresholds) | Minimal (platform updates) |
| **Suspect commit ID** | ✅ AI-powered | ❌ Manual | ✅ AI-powered |
| **Natural language reports** | ✅ Yes | ❌ No | ✅ Yes |
| **Deployment correlation** | ✅ Automatic | ✅ Automatic | ✅ Automatic |
| **Slow-growth detection** | ⚠️ Requires custom queries | ✅ Change Alerts | ⚠️ Requires custom queries |
| **Rate limits** | 50-150 req/day | None | API-specific |

**Recommendation:** Use AI-Powered approach (this doc) for best balance of intelligence and zero cost. Add Datadog Monitors as backup layer if desired.

---

## 10. Key Capabilities

### What AI Provides

**1. Intelligent Metric Selection:**
- AI analyzes commits to determine relevant metrics
- Example: Redis code changed → AI checks `redis.mem.used`, `redis.net.clients`
- No need to pre-configure "critical metrics" per service

**2. Context-Aware Analysis:**
- Not just "> 5% = alert" thresholds
- AI understands: "Error rate 0.1% → 0.3% (3x increase) is significant, but 5% → 6% (20% increase) might be normal variance"
- Considers deployment timing, service type, historical patterns

**3. Natural Language Explanations:**
- Instead of: "Error rate exceeded threshold"
- AI provides: "Error rate increased 192% correlating with payment validation changes in commit ghi789. Likely cause: strict validation logic rejecting edge cases."

**4. Automatic Suspect Commit Identification:**
- Cross-references metric anomalies with code changes
- Identifies most likely culprit commits
- Saves time in incident response

**5. Actionable Recommendations:**
- "Monitor closely for 1-2 hours"
- "Rollback recommended if error rate >0.5%"
- "All clear - metrics within normal range"

### Limitations to Be Aware Of

**GitHub Models API Rate Limits:**
- GPT-4o: 10 req/min, 50 req/day
- Llama-3.3-70B: 15 req/min, 150 req/day

**With 20 deployments/week:**
- Average: ~3 deployments/day
- Peak: Could hit 6-8 deployments/day
- Well within 50/day limit ✅

**If limits become an issue:**
- Use lighter Llama model (150 req/day)
- Add retry logic with exponential backoff
- Stagger deployments to avoid bursts

**AI Analysis Quality:**
- Depends on GitHub Models API uptime/quality
- Recommend: Keep Datadog Monitors as backup for critical alerts
- AI reports are supplementary intelligence, not safety-critical

---

## 11. Implementation Steps

### Phase 1: Prototype (Week 1)

**Day 1: Setup & Testing**
1. Create workflow template for AI deployment reports
2. Test with one pilot service (e.g., booking-api)
3. Validate end-to-end flow:
   - Deployment marked in Datadog ✓
   - Metrics queried successfully ✓
   - GitHub Models API call works ✓
   - Report posted to Lark ✓

**Day 2-3: Refinement**
1. Review AI report quality
2. Tune AI prompt for better insights
3. Adjust Datadog metric queries if needed
4. Test with 2-3 real deployments

**Day 4-5: Documentation**
1. Write setup guide for other services
2. Create secret management documentation
3. Document Lark webhook setup

### Phase 2: Rollout (Week 2)

**Day 1-2: Pilot Services**
1. Roll out to 3-5 pilot services
2. Add workflow to each service (15 min/service)
3. Configure secrets in GitHub repos
4. Test deployments

**Day 3-5: Monitor & Tune**
1. Review AI reports from pilot services
2. Identify false positives/negatives
3. Adjust AI prompt if needed
4. Document any issues/resolutions

### Phase 3: Full Rollout (Week 3)

**Day 1-3: All Services**
1. Roll out to remaining services
2. Batch secret configuration
3. Notify teams about new reporting

**Day 4-5: Daily Reports**
1. Implement daily summary workflow (optional)
2. Aggregate deployment reports from Lark
3. Post to Lark Doc at 6pm SGT

### Phase 4: Optional Enhancements (Week 4+)

**Datadog Monitors Backup Layer:**
1. Create critical threshold monitors
2. Configure as safety net for AI layer

**Slow-Growth Detection:**
1. Add memory/storage trend queries
2. Expand AI analysis to 1h/4h/24h windows

**Custom Metrics:**
1. Add service-specific metrics (Redis, cache hit rates, etc.)
2. Train AI with service-specific context

---

## Appendix A: Example AI Report Output

**Input Scenario:**
- Service: booking-api
- Deployment: v1.2.3
- Change: Added Redis caching + updated payment validation
- Error rate: 0.12% → 0.35%
- Latency: 245ms → 268ms

**AI-Generated Report:**

```markdown
## Deployment Report: booking-api v1.2.3
**Deployment Time:** 2026-05-13T14:32:00Z
**Status:** ⚠️ Warning - Elevated Error Rate

### Metrics Comparison
| Metric | Before | After | Change | Assessment |
|--------|--------|-------|--------|------------|
| Error Rate | 0.12% | 0.35% | +0.23pp (+192%) | ⚠️ Warning |
| Latency P95 | 245ms | 268ms | +23ms (+9%) | ✅ Normal |

### Analysis
Error rate nearly tripled from 0.12% to 0.35% following deployment. While the
absolute increase (+0.23 percentage points) is small, the 192% relative increase
is significant and correlates precisely with deployment timing.

Latency shows minor degradation (+9%) which falls within acceptable variance
and may be due to cold cache after deployment.

### Suspect Commits
**ghi789 - Update payment validation logic** (HIGH CONFIDENCE)
This commit modified payment validation rules. The error spike suggests the new
validation may be too strict or failing to handle certain edge cases that occur
in production but weren't covered in tests.

Recommended investigation:
- Review validation logic for edge cases
- Check error logs for specific validation failures
- Verify test coverage for payment validation paths

**abc123 - Add Redis caching for user sessions** (LOW CONFIDENCE)
Redis caching changes are unlikely to cause validation errors, but could
contribute if cache misses are causing unexpected state.

### Recommendation
⚠️ **Monitor closely for next 1-2 hours**

If error rate:
- Stays below 0.4%: Continue monitoring, investigate validation logic async
- Exceeds 0.5%: Prepare rollback plan
- Exceeds 1.0%: Rollback immediately

**Next steps:**
1. Query error logs for payment validation failures
2. Check if errors correlate with specific payment methods/regions
3. Consider hotfix to relax validation if blocking legitimate transactions
```

**Key Features Demonstrated:**
- ✅ Clear status indicator (⚠️ Warning)
- ✅ Quantitative comparison (both absolute and relative changes)
- ✅ Context-aware analysis (explains why 192% matters)
- ✅ Suspect commit identification with confidence levels
- ✅ Specific investigation steps
- ✅ Actionable thresholds for escalation

---

## Appendix B: Datadog API Authentication

**API Key vs App Key:**
- **API Key:** Used for sending data TO Datadog (events, metrics)
- **App Key:** Used for reading data FROM Datadog (queries, monitors)

**For this solution:**
- Only API Key needed (marking deployments)
- Get from: Datadog → Organization Settings → API Keys
- Store in: GitHub Secrets as `DATADOG_API_KEY`

**Security:**
- Keys don't auto-expire (manual rotation recommended)
- Rotate every 6-12 months
- Revoke immediately if compromised

---

## 12. PoC (Proof of Concept) - Executable Plan

### 📚 参考案例：Cursor Agent集成Pattern

**发现：Traveloka已有成功案例** - `cwang215/fpr-fprfb` 使用reusable workflow集成Copilot/Cursor agent

**Pattern:**
```yaml
# Trigger workflow
jobs:
  cursor-npe:
    uses: cwang215/fpr-fprfb/.github/workflows/center-npe-cursor.yml@master
    with:
      issue_number: ${{ github.event.issue.number }}
      # ... other inputs ...
    secrets:
      CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
      # ... other secrets ...
```

**关键点：**
- ✅ 使用reusable workflow（和CI/CD一致）
- ✅ 参数通过 `with:` 传递
- ✅ Secrets集中管理
- ✅ 调用者只需简单配置

**我们将采用完全相同的pattern！**

---

**This PoC is 100% safe and will NOT impact any existing functionality:**

✅ **Separate workflow file** - Creates NEW file `deployment-report-poc.yaml`, does NOT modify existing files
✅ **Manual trigger only** - Uses `workflow_dispatch` (manual button click), does NOT auto-run
✅ **Runs AFTER deployment** - Only queries metrics post-deployment, does NOT touch deployment process
✅ **Read-only operations** - Only reads from Datadog, does NOT write (except harmless event markers)
✅ **Independent execution** - Runs in separate GitHub Actions job, no dependencies on deployment
✅ **Failure isolation** - If PoC fails, deployment is already complete and unaffected

**What happens if PoC fails?**
- ❌ AI report doesn't post to Lark → **No impact on production**
- ❌ Datadog query fails → **No impact on production**
- ❌ GitHub Models API errors → **No impact on production**
- ✅ **Production deployment already succeeded before PoC starts**

**What existing files are modified?**
- **NONE** - PoC creates a NEW workflow file only

**What secrets are needed?**
- Datadog API keys (read-only usage)
- Lark webhook (just posts messages)
- All are scoped to the PoC workflow only

---

### PoC Goal
Validate the entire AI deployment report flow on ONE service **without modifying shared platform, generated files, or ANY existing workflows**.

### Risk Assessment

| Risk | Mitigation | Impact if PoC Fails |
|------|-----------|---------------------|
| PoC workflow fails | Runs AFTER deployment completes | Zero - deployment already succeeded |
| Datadog query errors | Read-only, no writes to metrics | Zero - just queries, no changes |
| GitHub Models API rate limit | Manual trigger, controlled testing | Zero - only affects PoC reports |
| Lark notification fails | Only informational | Zero - no production impact |
| Wrong metrics queried | PoC is manual, can be tested offline | Zero - doesn't affect deployment |

**Blast radius: ZERO** - PoC is completely isolated from production deployment flow.

### PoC Implementation (Minimal Risk)

**Approach: 使用和Cursor Agent一样的Reusable Workflow Pattern**

#### Option A: PoC with Standalone Workflow (推荐用于测试)

**Step 1: Create PoC Reusable Workflow** (10 minutes)

Create in a test/shared repo (e.g., your personal fork or `bei-monitoring` if you have access):

**File:** `.github/workflows/deployment-report-poc.yaml`

```yaml
name: AI Deployment Report (PoC Reusable)

on:
  workflow_call:
    inputs:
      service_name:
        required: true
        type: string
      deployment_time:
        required: true
        type: string
      commit_sha:
        required: true
        type: string
      environment:
        required: false
        type: string
        default: 'production'
    secrets:
      DATADOG_API_KEY:
        required: true
      DATADOG_APP_KEY:
        required: true
      LARK_WEBHOOK:
        required: true

permissions:
  contents: read

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ inputs.commit_sha }}
          fetch-depth: 20
      
      - name: Wait for Metrics
        run: |
          echo "⏰ Waiting 15 minutes for metrics..."
          sleep 900
      
      - name: Query Datadog Metrics
        id: metrics
        env:
          DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
          DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
          DEPLOYMENT_TIME: ${{ inputs.deployment_time }}
          SERVICE_NAME: ${{ inputs.service_name }}
        run: |
          # Calculate time windows
          BEFORE_START=$(date -u -d "$DEPLOYMENT_TIME - 1 hour" +%s)
          BEFORE_END=$(date -u -d "$DEPLOYMENT_TIME" +%s)
          AFTER_START=$BEFORE_END
          AFTER_END=$(date -u +%s)
          
          echo "📊 Querying Datadog metrics..."
          
          # Query error rate
          BEFORE_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:$SERVICE_NAME,env:production}" \
            -d "from=$BEFORE_START" -d "to=$BEFORE_END")
          
          AFTER_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:$SERVICE_NAME,env:production}" \
            -d "from=$AFTER_START" -d "to=$AFTER_END")
          
          # Save metrics
          echo "$BEFORE_ERRORS" > before.json
          echo "$AFTER_ERRORS" > after.json
          
          echo "✅ Metrics queried successfully"
      
      - name: Generate AI Report with GitHub Models
        env:
          GITHUB_TOKEN: ${{ github.token }}
          SERVICE_NAME: ${{ inputs.service_name }}
          COMMIT_SHA: ${{ inputs.commit_sha }}
          DEPLOYMENT_TIME: ${{ inputs.deployment_time }}
        run: |
          # Get commits
          COMMITS=$(git log --oneline --no-decorate -10)
          
          # Read metrics
          BEFORE=$(cat before.json)
          AFTER=$(cat after.json)
          
          # Create AI prompt
          AI_PROMPT="You are an SRE analyzing a production deployment.

Service: $SERVICE_NAME
Commit: $COMMIT_SHA
Time: $DEPLOYMENT_TIME

Recent commits:
$COMMITS

Datadog Metrics:
Before (1h avg): $BEFORE
After (15min avg): $AFTER

Parse the JSON, compare metrics, and generate a deployment health report.

Format:
## Deployment Report: $SERVICE_NAME
**Status:** ✅/⚠️/❌
**Time:** $DEPLOYMENT_TIME
**Commit:** $COMMIT_SHA

### Metrics
| Metric | Before | After | Change | Status |
|--------|--------|-------|--------|--------|

### Analysis
[Brief analysis]

### Recommendation
[Action needed]"

          # Call GitHub Models API
          echo "🤖 Calling GitHub Models API..."
          AI_REPORT=$(curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $GITHUB_TOKEN" \
            -d '{
              "model": "gpt-4o",
              "messages": [
                {"role": "system", "content": "You are an SRE expert. Be concise."},
                {"role": "user", "content": "'"${AI_PROMPT}"'"}
              ],
              "max_tokens": 1500
            }' | jq -r '.choices[0].message.content // "Error: AI call failed"')
          
          echo "$AI_REPORT" > report.md
          cat report.md
      
      - name: Post to Lark
        env:
          LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
          SERVICE_NAME: ${{ inputs.service_name }}
        run: |
          REPORT=$(cat report.md | jq -Rs .)
          
          curl -X POST "$LARK_WEBHOOK" \
            -H "Content-Type: application/json" \
            -d '{
              "msg_type": "interactive",
              "card": {
                "header": {
                  "title": {"content": "📊 PoC Report: '"$SERVICE_NAME"'", "tag": "plain_text"},
                  "template": "blue"
                },
                "elements": [
                  {"tag": "markdown", "content": '"$REPORT"'}
                ]
              }
            }'
          
          echo "✅ Report posted to Lark"
```

---

**Step 2: Create Caller Workflow in Service Repo** (5 minutes)

In `fpr-fprmfdt/.github/workflows/deployment-report-poc-caller.yaml`:

```yaml
name: PoC - Call AI Report

on:
  workflow_dispatch:
    inputs:
      deployment_time:
        description: 'Deployment time (ISO 8601)'
        required: true
        type: string
      commit_sha:
        description: 'Deployed commit SHA'
        required: true
        type: string

jobs:
  call-ai-report:
    # 调用reusable workflow (和Cursor pattern一样)
    uses: jinjinwang11/ai-release-monitoring/.github/workflows/deployment-report-poc.yaml@main
    with:
      service_name: fprmfdt
      deployment_time: ${{ inputs.deployment_time }}
      commit_sha: ${{ inputs.commit_sha }}
      environment: production
    secrets:
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
      DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
      LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
```

**Key Benefits of This Approach:**
- ✅ 和Cursor集成pattern完全一致
- ✅ Reusable workflow可以被所有service复用
- ✅ Service repo只需简单的caller workflow
- ✅ 集中管理和更新（只改reusable workflow）

---

#### Option B: Standalone PoC (如果没有共享repo)

如果暂时没有共享repo，可以先在service自己的repo测试：

```yaml
name: AI Deployment Report (PoC)

on:
  workflow_dispatch:
    inputs:
      deployment_time:
        description: 'Deployment time (ISO 8601, e.g., 2026-05-13T14:32:00Z)'
        required: true
        type: string
      commit_sha:
        description: 'Deployed commit SHA'
        required: true
        type: string

permissions:
  contents: read
  models: read

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ inputs.commit_sha }}
          fetch-depth: 20
      
      - name: Wait for Metrics Stabilization
        run: |
          echo "Waiting 15 minutes for metrics..."
          sleep 900
      
      - name: Mark Deployment in Datadog
        run: |
          curl -X POST "https://api.datadoghq.com/api/v1/events" \
            -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{
              "title": "Deployment: fprmfdt PoC",
              "text": "PoC test deployment for commit ${{ inputs.commit_sha }}",
              "tags": ["deployment", "service:fprmfdt", "env:production", "poc"],
              "date_happened": '"$(date -d "${{ inputs.deployment_time }}" +%s)"'
            }'
      
      - name: Query Datadog Metrics
        id: metrics
        env:
          DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
          DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
          DEPLOYMENT_TIME: ${{ inputs.deployment_time }}
        run: |
          # Time windows
          BEFORE_START=$(date -u -d "$DEPLOYMENT_TIME - 1 hour" +%s)
          BEFORE_END=$(date -u -d "$DEPLOYMENT_TIME" +%s)
          AFTER_START=$BEFORE_END
          AFTER_END=$(date -u +%s)
          
          echo "Querying metrics..."
          echo "Before window: $(date -d @$BEFORE_START) to $(date -d @$BEFORE_END)"
          echo "After window: $(date -d @$AFTER_START) to $(date -d @$AFTER_END)"
          
          # Query error rate before
          BEFORE_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:fprmfdt,env:production}" \
            -d "from=$BEFORE_START" \
            -d "to=$BEFORE_END")
          
          # Query error rate after
          AFTER_ERRORS=$(curl -G "https://api.datadoghq.com/api/v1/query" \
            -H "DD-API-KEY: $DATADOG_API_KEY" \
            -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \
            --data-urlencode "query=avg:trace.servlet.request.errors{service:fprmfdt,env:production}" \
            -d "from=$AFTER_START" \
            -d "to=$AFTER_END")
          
          # Save to file for next step
          echo "$BEFORE_ERRORS" > before_errors.json
          echo "$AFTER_ERRORS" > after_errors.json
          
          echo "Metrics queried successfully"
      
      - name: Generate AI Report
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          DEPLOYMENT_TIME: ${{ inputs.deployment_time }}
          COMMIT_SHA: ${{ inputs.commit_sha }}
        run: |
          # Get commit messages
          COMMITS=$(git log --oneline --no-decorate -10)
          
          # Read metric data
          BEFORE_ERRORS=$(cat before_errors.json)
          AFTER_ERRORS=$(cat after_errors.json)
          
          # Create AI prompt
          AI_PROMPT="You are an SRE analyzing a production deployment.

Service: fprmfdt
Commit: $COMMIT_SHA
Deployment Time: $DEPLOYMENT_TIME

Recent commits:
$COMMITS

Datadog Metrics:
Before deployment (1h avg): $BEFORE_ERRORS
After deployment (15min avg): $AFTER_ERRORS

Task: Parse the JSON, compare metrics, identify any issues, and generate a deployment health report.

Format:
## Deployment Report: fprmfdt
**Status:** [✅ Healthy / ⚠️ Warning / ❌ Critical]
**Deployment Time:** $DEPLOYMENT_TIME
**Commit:** $COMMIT_SHA

### Metrics
| Metric | Before | After | Change | Status |
|--------|--------|-------|--------|--------|
| Error Rate | X | Y | Z | ... |

### Analysis
[Brief analysis]

### Recommendation
[Action needed]"

          # Call GitHub Models API
          echo "Calling GitHub Models API..."
          AI_REPORT=$(curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $GITHUB_TOKEN" \
            -d '{
              "model": "gpt-4o",
              "messages": [
                {"role": "system", "content": "You are an SRE expert. Be concise."},
                {"role": "user", "content": "'"${AI_PROMPT}"'"}
              ],
              "max_tokens": 1500
            }' | jq -r '.choices[0].message.content // "Error: AI call failed"')
          
          echo "$AI_REPORT" > report.md
          cat report.md
      
      - name: Post to Lark
        env:
          LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
        run: |
          REPORT=$(cat report.md | jq -Rs .)
          
          curl -X POST "$LARK_WEBHOOK" \
            -H "Content-Type: application/json" \
            -d '{
              "msg_type": "interactive",
              "card": {
                "header": {
                  "title": {"content": "📊 PoC Deployment Report: fprmfdt", "tag": "plain_text"},
                  "template": "blue"
                },
                "elements": [
                  {"tag": "markdown", "content": '"$REPORT"'}
                ]
              }
            }'
          
          echo "✅ Report posted to Lark"
```

**Step 2: Configure Secrets** (5 minutes)

In `fpr-fprmfdt` repo settings → Secrets and variables → Actions, add:
- `DATADOG_API_KEY` - Get from Datadog → Organization Settings → API Keys
- `DATADOG_APP_KEY` - Get from Datadog → Organization Settings → Application Keys
- `LARK_WEBHOOK` - Your Lark webhook URL

**Step 3: Test the PoC** (2 hours)

**Timeline:**
1. ✅ Create workflow file (5 min)
2. ✅ Configure secrets (5 min)
3. ⏳ **Wait for next production deployment** of fprmfdt (variable timing)
4. ✅ **After deployment completes successfully** (verify first!)
5. ✅ Note deployment time and commit SHA from GitHub Actions
6. ✅ **Manually trigger PoC workflow** (not automatic!)
   - Go to: `https://github.com/traveloka/fpr-fprmfdt/actions/workflows/deployment-report-poc.yaml`
   - Click "Run workflow"
   - Enter deployment time (from step 5)
   - Enter commit SHA (from step 5)
   - Click "Run workflow"
7. ⏳ **Wait ~16 minutes** (15 min sleep + 1 min execution)
8. ✅ Check Lark for the AI report

**Important:**
- ⚠️ **Only trigger AFTER production deployment succeeds**
- ⚠️ PoC workflow is separate - it doesn't run during deployment
- ⚠️ If you trigger with wrong time/SHA, no harm - just bad report data

**Step 4: Validate** (30 minutes)

✅ Does the report appear in Lark?
✅ Are metrics correctly queried from Datadog?
✅ Does AI analysis make sense?
✅ Does GitHub Models API work?

**Step 5: Iterate** (if needed)

- Adjust AI prompt for better reports
- Tune metric queries
- Add more metrics (latency, throughput, etc.)

### Next Steps After PoC Success

Once validated:
1. Convert to automated (trigger after deployment succeeds)
2. Integrate with `bei-backend-ci-cd-platform`
3. Rollout to other services

### Estimated Time
- Setup: 20 minutes
- Wait for test deployment: Variable
- Validation: 30 minutes
- **Total: < 1 hour of actual work**

---
