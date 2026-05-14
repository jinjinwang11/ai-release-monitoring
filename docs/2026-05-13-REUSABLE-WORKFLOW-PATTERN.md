# Reusable Workflow Pattern - 基于Cursor Agent案例

## 为什么使用这个Pattern？

Traveloka内部已经有成功案例：`cwang215/fpr-fprfb` 用reusable workflow集成Cursor agent

**优势：**
✅ 和现有CI/CD pattern一致
✅ 和Cursor agent integration一致  
✅ 集中管理 - 一个地方维护
✅ 易于rollout - 每个service只需简单调用

---

## Pattern对比

### Cursor Agent Pattern (已验证)
```yaml
# fpr-fprfb/.github/workflows/center-npe-cursor.yml
jobs:
  cursor-npe:
    uses: cwang215/fpr-fprfb/.github/workflows/center-npe-cursor.yml@master
    with:
      issue_number: ${{ github.event.issue.number }}
      issue_title: ${{ github.event.issue.title }}
    secrets:
      CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
```

### Our AI Report Pattern (相同结构)
```yaml
# fpr-fprmfdt/.github/workflows/deployment-report-caller.yml
jobs:
  ai-report:
    uses: traveloka/bei-monitoring/.github/workflows/deployment-report.yml@v1
    with:
      service_name: fprmfdt
      deployment_time: ${{ github.event.head_commit.timestamp }}
    secrets:
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
```

---

## 实施步骤（基于Proven Pattern）

### Step 1: 创建Reusable Workflow
**Location:** `traveloka/bei-monitoring/.github/workflows/deployment-report.yml`

```yaml
name: AI Deployment Report (Reusable)

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
    secrets:
      DATADOG_API_KEY:
        required: true
      DATADOG_APP_KEY:
        required: true
      LARK_WEBHOOK:
        required: true

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      # ... 实际的报告生成逻辑 ...
```

### Step 2: 在Service中调用
**Location:** `fpr-fprmfdt/.github/workflows/fprmfdt_ecs.yaml` (或者新建caller)

```yaml
jobs:
  standard_deployment:
    # ... 现有部署 ...
  
  ai-report:
    needs: standard_deployment
    if: success()
    uses: traveloka/bei-monitoring/.github/workflows/deployment-report.yml@v1
    with:
      service_name: fprmfdt
      deployment_time: ${{ github.event.head_commit.timestamp }}
      commit_sha: ${{ github.sha }}
    secrets: inherit  # 或者明确传递secrets
```

---

## PoC测试方式

### Option A: 创建独立测试workflow (推荐)
```yaml
# fpr-fprmfdt/.github/workflows/poc-ai-report.yml
name: PoC AI Report Test

on:
  workflow_dispatch:
    inputs:
      deployment_time:
        required: true
        type: string
      commit_sha:
        required: true
        type: string

jobs:
  test:
    uses: YOUR_FORK/bei-monitoring/.github/workflows/deployment-report.yml@main
    with:
      service_name: fprmfdt
      deployment_time: ${{ inputs.deployment_time }}
      commit_sha: ${{ inputs.commit_sha }}
    secrets:
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
      DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
      LARK_WEBHOOK: ${{ secrets.LARK_WEBHOOK }}
```

**测试流程：**
1. 等下一次fprmfdt部署完成
2. 手动trigger这个PoC workflow
3. 输入deployment time和commit SHA
4. 验证报告质量

---

## 和现有Integration对比

| 集成类型 | Pattern | 文件位置 |
|---------|---------|---------|
| CI Test | Reusable workflow | `bei-backend-ci-cd-platform/ci_build.yaml` |
| Deployment | Reusable workflow | `bei-backend-ci-cd-platform/ecs_deployment.yaml` |
| Cursor Agent | Reusable workflow | `fpr-fprfb/center-npe-cursor.yml` |
| **AI Report** | **Reusable workflow** | **`bei-monitoring/deployment-report.yml`** |

**完全一致的pattern！** ✅

---

## 后续Rollout

1. **PoC阶段:** 手动trigger测试
2. **Pilot阶段:** 2-3个service自动调用
3. **Rollout阶段:** 
   - Option A: 修改`bei-backend-ci-cd-platform/ecs_deployment.yaml`，所有service自动获得
   - Option B: 更新`generateBflowEcs` template，自动为所有service生成调用

