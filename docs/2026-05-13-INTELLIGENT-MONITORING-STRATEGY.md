# 智能Monitoring策略设计

## 两层监控策略

### Layer 1: Hard Criteria (Datadog Monitors)
**特点：** 固定阈值，适用于所有deployment
**用途：** Safety net - 防止严重问题
**技术：** Datadog Anomaly Detection Monitors

**示例：**
```
✅ Error rate > 5% → 立即报警
✅ Latency P95 > 1000ms → 立即报警
✅ CPU usage > 90% → 立即报警
✅ Memory leak (1h vs 4h增长 >30%) → 报警
```

**优势：**
- 不需要AI，基于历史数据学习
- 即时响应，Datadog直接alert
- 独立于AI report运行

---

### Layer 2: Commit-Based Intelligent Monitoring (AI)
**特点：** 根据代码改动动态选择metrics
**用途：** 深度分析 - 发现subtle问题
**技术：** AI分析commits + 动态查询Datadog

## 实现方案

### 方案A: AI分析Commit选择Metrics (推荐)

**流程：**
```
1. 获取deployment commits
   ↓
2. AI分析代码改动：
   - 改了Redis代码 → 监控 redis.mem.used, redis.connections
   - 改了Database query → 监控 db.query.duration, db.connections
   - 改了Cache逻辑 → 监控 cache.hit_rate, cache.miss_rate
   - 改了API endpoint → 监控该endpoint的latency/errors
   ↓
3. 动态构建Datadog查询
   ↓
4. AI对比分析 before/after
   ↓
5. 生成智能报告
```

**实现示例：**

```yaml
- name: AI-Powered Metric Selection
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    COMMIT_SHA: ${{ inputs.commit_sha }}
  run: |
    # Get commit diff
    DIFF=$(git diff $COMMIT_SHA~10..$COMMIT_SHA)
    
    # AI分析：哪些metrics应该监控？
    AI_PROMPT="Analyze this code diff and identify which metrics to monitor.

Code changes:
$DIFF

Return a JSON array of metrics to monitor. Format:
[
  {
    \"metric\": \"redis.mem.used\",
    \"reason\": \"Redis caching code added\",
    \"threshold\": \"growth >20%\"
  },
  {
    \"metric\": \"trace.servlet.request.errors{endpoint:/api/booking}\",
    \"reason\": \"Modified booking endpoint validation\",
    \"threshold\": \"increase >50%\"
  }
]"

    METRICS_TO_MONITOR=$(curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
      -H "Authorization: Bearer $GITHUB_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You are an SRE expert. Return ONLY valid JSON."},
          {"role": "user", "content": "'"$AI_PROMPT"'"}
        ],
        "response_format": {"type": "json_object"}
      }' | jq -r '.choices[0].message.content')
    
    echo "$METRICS_TO_MONITOR" > metrics_to_monitor.json
```

**然后动态查询这些metrics：**

```bash
- name: Query Dynamic Metrics
  run: |
    # Read AI-selected metrics
    METRICS=$(cat metrics_to_monitor.json | jq -r '.[]')
    
    # For each metric, query Datadog
    for metric in $(echo "$METRICS" | jq -r '.metric'); do
      BEFORE=$(curl ... "query=$metric...")
      AFTER=$(curl ... "query=$metric...")
      # Compare and analyze
    done
```

---

### 方案B: 基于文件路径的Smart Rules

**规则引擎：**
```yaml
# smart_monitoring_rules.yml
rules:
  - pattern: "*/redis/*"
    metrics:
      - redis.mem.used
      - redis.connections
      - redis.evicted_keys
    
  - pattern: "*/database/*"
    metrics:
      - db.query.duration
      - db.connections.active
      - db.slow_queries
    
  - pattern: "*/cache/*"
    metrics:
      - cache.hit_rate
      - cache.miss_rate
      - cache.evictions
    
  - pattern: "*/payment/*"
    metrics:
      - trace.servlet.request.errors{service:payment}
      - payment.transaction.duration
      - payment.gateway.errors
```

**实现：**
```bash
# 分析changed files
CHANGED_FILES=$(git diff --name-only $COMMIT_SHA~10..$COMMIT_SHA)

# 匹配rules，确定要监控的metrics
for file in $CHANGED_FILES; do
  # Match against rules
  if [[ $file == *redis* ]]; then
    METRICS+="redis.mem.used redis.connections "
  fi
  if [[ $file == *database* ]]; then
    METRICS+="db.query.duration db.connections "
  fi
done

# Query those metrics
```

---

### 方案C: Hybrid (最佳实践)

**结合三种方式：**

```
1. Hard Criteria (Datadog Monitors)
   → 立即alert critical issues
   ↓
2. Smart Rules (File-based)
   → 快速确定baseline metrics
   ↓
3. AI Dynamic Analysis
   → 深度分析code changes，发现subtle issues
```

**示例流程：**

```yaml
- name: Multi-Layer Monitoring
  run: |
    # Layer 1: Datadog monitors (already running, separate)
    echo "✅ Datadog monitors active"
    
    # Layer 2: Smart rules - baseline metrics
    BASELINE_METRICS="trace.servlet.request.errors trace.servlet.request.duration.95p"
    
    # Layer 3: AI analysis - additional metrics
    DIFF=$(git diff --name-only $COMMIT_SHA~10..$COMMIT_SHA)
    
    AI_ADDITIONAL_METRICS=$(curl ... AI analysis of diff ...)
    
    # Combine all metrics
    ALL_METRICS="$BASELINE_METRICS $SMART_RULE_METRICS $AI_ADDITIONAL_METRICS"
    
    # Query and analyze
    for metric in $ALL_METRICS; do
      # Query before/after
      # AI comparison and reporting
    done
```

---

## 具体示例

### 场景1: Redis Cache添加

**Commit:**
```java
// Added Redis caching
@Cacheable(value = "userSessions", key = "#userId")
public UserSession getUserSession(String userId) {
    return database.query(...);
}
```

**AI自动识别：**
```json
{
  "additional_metrics": [
    "redis.mem.used",
    "redis.net.clients", 
    "cache.hit_rate{cache:userSessions}"
  ],
  "monitoring_focus": "Memory growth and cache effectiveness",
  "alert_if": "redis.mem.used increases >30% in 1h"
}
```

**报告：**
```markdown
### Redis Monitoring (Code Change Detected)
**Change:** Added caching for user sessions

| Metric | Before | After | Status |
|--------|--------|-------|--------|
| redis.mem.used | 500MB | 520MB | ✅ Normal (+4%) |
| cache.hit_rate | N/A | 85% | ✅ Good |

**Analysis:** Cache working as expected, memory impact minimal.
```

---

### 场景2: Payment Validation Logic改动

**Commit:**
```java
// Updated payment validation
if (amount > 10000 && !user.isVerified()) {
    throw new ValidationException("High amount requires verification");
}
```

**AI自动识别：**
```json
{
  "additional_metrics": [
    "trace.servlet.request.errors{endpoint:/payment/validate}",
    "payment.validation.failures",
    "payment.high_amount_rejections"
  ],
  "monitoring_focus": "Validation rejection rate",
  "alert_if": "validation errors increase >100%"
}
```

**报告：**
```markdown
### Payment Validation Monitoring (Code Change Detected)
**Change:** Added verification requirement for high-value transactions

| Metric | Before | After | Status |
|--------|--------|-------|--------|
| validation errors | 0.1% | 0.8% | ⚠️ Warning (+700%) |
| high_amount_rejections | N/A | 12/hour | ℹ️ New metric |

**Analysis:** Error rate spike correlates with new validation. 
**Suspect Commit:** abc123 - Added verification check
**Recommendation:** Review if 10000 threshold is too low, causing 
legitimate transactions to be rejected.
```

---

## 实现优先级

### Phase 1: Hard Criteria (已有方案)
- Datadog Anomaly Detection Monitors
- 立即可用，安全网

### Phase 2: Baseline + AI Analysis (PoC)
- Baseline metrics (error, latency) + AI报告
- 这是当前PoC方案

### Phase 3: Commit-Based Smart Monitoring
- AI分析commits → 动态选择metrics
- 更智能，但需要更多开发

### Phase 4: Hybrid System
- 三层结合：Hard + Smart Rules + AI
- 最完整的solution

