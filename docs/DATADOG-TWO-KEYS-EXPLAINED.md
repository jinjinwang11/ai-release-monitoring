# Datadog API Key vs Application Key

## 🔑 为什么需要两个Key？

Datadog使用**双重认证**机制来保护API访问：

### API Key (Organization-wide)
- **用途：** 识别你的**组织**
- **作用域：** 整个Datadog organization
- **类比：** 像公司的门禁卡（证明你是这个公司的）

### Application Key (User-specific)
- **用途：** 识别**具体的用户/应用**
- **作用域：** 特定用户的权限
- **类比：** 像你的工牌（证明你是谁，有什么权限）

---

## 📊 实际使用场景

### 场景1: 创建Event（只需API Key）✅

```bash
# Mark deployment in Datadog
curl -X POST "https://api.datadoghq.com/api/v1/events" \
  -H "DD-API-KEY: $DATADOG_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "Deployment", "text": "..."}'
```

**只需要API Key因为：**
- ✅ 写入event是组织级操作
- ✅ 不需要特定用户权限

---

### 场景2: 查询Metrics（需要两个Key）⚠️

```bash
# Query metrics
curl -G "https://api.datadoghq.com/api/v1/query" \
  -H "DD-API-KEY: $DATADOG_API_KEY" \           # 识别组织
  -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \   # 识别用户权限
  --data-urlencode "query=avg:trace.servlet.request.errors{...}" \
  -d "from=$BEFORE_START" \
  -d "to=$BEFORE_END"
```

**需要两个Key因为：**
- 🔐 查询metrics是**敏感操作**（读取生产数据）
- 🔐 需要验证**用户有权限**访问这些metrics
- 🔐 Datadog需要知道**谁在查询**（audit log）

---

## 🎯 我们的Workflow使用情况

### Step 1: Mark Deployment（只用API Key）

```yaml
- name: Mark Deployment in Datadog
  run: |
    curl -X POST "https://api.datadoghq.com/api/v1/events" \
      -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
      -d '{"title": "Deployment: fprmfdt", ...}'
```

**OK，只需要API Key！** ✅

---

### Step 2: Query Metrics（需要两个Key）

```yaml
- name: Query Datadog Metrics
  env:
    DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
    DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}  # 必需！
  run: |
    # Query error rate
    curl -G "https://api.datadoghq.com/api/v1/query" \
      -H "DD-API-KEY: $DATADOG_API_KEY" \
      -H "DD-APPLICATION-KEY: $DATADOG_APP_KEY" \  # 没有这个会401!
      --data-urlencode "query=..."
```

**必须有App Key，否则：**
```json
{
  "errors": ["Forbidden: Application key required"]
}
```

---

## 🔐 安全设计原因

### 为什么Datadog这么设计？

**问题：** 如果只用一个key会怎样？

**风险1: API Key泄露**
- 如果只用API Key，泄露后别人可以：
  - ❌ 读取所有metrics（敏感数据）
  - ❌ 查询所有dashboards
  - ❌ 导出所有数据

**风险2: 无法追踪谁在操作**
- 所有操作都是"组织"做的
- ❌ 无法审计谁查询了什么
- ❌ 无法限制特定用户权限

---

**双Key设计的好处：**

1. **权限分离：**
   - API Key = 组织身份（轻量级，可以公开一些）
   - App Key = 用户权限（严格控制，个人专属）

2. **精细控制：**
   - 可以给不同App Key不同权限
   - 某个App Key只能读metrics，不能写
   - 另一个App Key可以创建monitors

3. **审计追踪：**
   - Datadog知道是哪个App Key查询的
   - 可以追踪谁访问了敏感数据
   - 方便compliance审计

4. **泄露风险降低：**
   - API Key泄露 → 只能写events（影响小）
   - App Key泄露 → 只撤销这一个key，不影响组织

---

## 📋 Datadog API 权限矩阵

| API Endpoint | API Key | App Key | 说明 |
|--------------|---------|---------|------|
| POST /events | ✅ | ❌ | 创建event，只需组织身份 |
| POST /metrics | ✅ | ❌ | 提交metrics，只需组织身份 |
| GET /query | ✅ | ✅ | **查询metrics，需要用户权限** |
| GET /dashboards | ✅ | ✅ | 查询dashboards |
| GET /monitors | ✅ | ✅ | 查询monitors |
| POST /monitors | ✅ | ✅ | 创建monitors（需要权限） |
| GET /logs | ✅ | ✅ | **查询logs，需要用户权限** |

**规律：**
- **写入数据**（POST metrics/events）→ 只需API Key
- **读取数据**（GET query/logs）→ 需要API Key + App Key

---

## 🤔 常见问题

### Q1: 能不能只用一个key？

**A:** 不行。对于我们的use case（查询metrics），Datadog **强制要求** App Key。

**试试看会怎样：**
```bash
# 只用API Key查询metrics
curl -G "https://api.datadoghq.com/api/v1/query" \
  -H "DD-API-KEY: $DATADOG_API_KEY" \
  --data-urlencode "query=..."

# 返回：
{
  "errors": ["Forbidden: Application key required"]
}
```

---

### Q2: App Key是每个人都不一样的吗？

**A:** 可以！有两种方式：

**方式1: 个人App Key**
- 每个Datadog用户创建自己的App Key
- 好处：审计清楚（知道是谁查的）
- 坏处：离职了需要更换

**方式2: Service Account App Key（推荐）**
- 创建一个"AI Monitoring"服务账号
- 给它一个专用的App Key
- 好处：不依赖个人，长期稳定
- Datadog推荐用于automation

---

### Q3: 这两个key会过期吗？

**A:** 
- **API Key:** 不会自动过期（除非手动revoke）
- **App Key:** 不会自动过期（除非手动revoke或用户离职被删）

**建议：**
- 定期轮换（6-12个月）
- 用org-level secrets存储（方便轮换）
- 记录创建时间（Datadog UI显示）

---

### Q4: 权限可以限制吗？

**A:** App Key可以限制scope！

创建App Key时可以指定：
```
Scopes:
  ✅ metrics_read      - 可以查询metrics
  ✅ events_read       - 可以读events
  ❌ monitors_write    - 不能创建monitors
  ❌ dashboards_write  - 不能修改dashboards
```

**我们只需要：**
- `metrics_read` - 查询metrics
- `events_write` - 创建deployment markers（可选）

---

## ✅ 总结

### 为什么需要两个Key？

| 原因 | 说明 |
|------|------|
| **安全设计** | 双重认证，降低泄露风险 |
| **权限控制** | App Key可以限制具体权限 |
| **审计追踪** | 知道谁查询了什么数据 |
| **Datadog强制** | 查询metrics API必须提供App Key |

### 类比理解

```
银行转账需要：
  1. 银行卡（API Key）     - 证明你是这个银行的客户
  2. 密码（App Key）        - 证明你有权限操作这个账户

Datadog查询需要：
  1. API Key               - 证明你的组织在Datadog
  2. Application Key       - 证明你有权限查询这些数据
```

### 我们的使用

```yaml
secrets:
  DATADOG_API_KEY: "xxxx"     # 组织身份（所有Datadog操作都需要）
  DATADOG_APP_KEY: "yyyy"     # 用户权限（查询metrics/logs必需）
```

**都需要！缺一不可！** ✅

---

## 📖 参考资料

- [Datadog API Authentication](https://docs.datadoghq.com/api/latest/authentication/)
- [Datadog Application Keys](https://docs.datadoghq.com/account_management/api-app-keys/)
- [API vs App Keys](https://docs.datadoghq.com/account_management/api-app-keys/#api-keys)
