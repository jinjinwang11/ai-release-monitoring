# Self-Test Guide for ai-release-monitoring

## 🧪 Testing the Workflow in This Repo

You can test the reusable workflow directly in this repo without touching any service repos!

---

## ✅ Setup (One-time)

### Step 1: Configure Secrets in THIS Repo

1. **Go to this repo's settings:**
   ```
   https://github.com/jinjinwang11/ai-release-monitoring/settings/secrets/actions
   ```

2. **Create 3 secrets:**

   | Secret Name | Value | Where to Get |
   |-------------|-------|--------------|
   | `DATADOG_API_KEY` | Your Datadog API key | [Datadog → Organization Settings → API Keys](https://app.datadoghq.com/organization-settings/api-keys) |
   | `DATADOG_APP_KEY` | Your Datadog App key | [Datadog → Organization Settings → Application Keys](https://app.datadoghq.com/organization-settings/application-keys) |
   | `LARK_WEBHOOK` | Your Lark webhook URL | Lark → Channel Settings → Bots → Add Custom Bot |

---

## 🚀 Run Test

### Step 1: Trigger Test Workflow

1. **Go to Actions:**
   ```
   https://github.com/jinjinwang11/ai-release-monitoring/actions/workflows/test-self.yaml
   ```

2. **Click "Run workflow"**

3. **Fill in test parameters:**
   - **deployment_time**: Use a recent deployment time
     - Example: `2026-05-14T06:00:00Z` (6am UTC = 2pm Beijing)
   - **commit_sha**: Use a recent commit from any service
     - Example: `abc123def456` (from fpr-fprmfdt)
   - **service_name**: Service to query metrics from
     - Example: `fprmfdt`

4. **Click "Run workflow"** (green button)

---

## ⏰ Wait for Results

The workflow will:
1. ⏰ Sleep 15 minutes (configurable)
2. 📊 Query Datadog for the service metrics
3. 🤖 Generate AI analysis
4. 📤 Post to Lark

**Total time:** ~16 minutes

---

## ✅ Verify

### Check GitHub Actions

1. Go to: [Actions](https://github.com/jinjinwang11/ai-release-monitoring/actions)
2. Find your test run
3. Check all steps completed successfully
4. Look for "✅ Report posted to Lark"

### Check Lark

1. Open your configured Lark channel
2. You should see a blue card with:
   - Service name
   - Metrics table
   - AI analysis
   - Recommendations

### Download Artifacts

1. In GitHub Actions run page
2. Scroll to "Artifacts"
3. Download report files:
   - `report.md` - AI-generated report
   - `*.json` - Raw Datadog metrics

---

## 🐛 Troubleshooting

### Error: "Secret not found"

**Fix:**
1. Verify secrets are created in **this repo** settings
2. Check secret names exactly match:
   - `DATADOG_API_KEY`
   - `DATADOG_APP_KEY`
   - `LARK_WEBHOOK`

### Error: "Datadog API failed"

**Fix:**
1. Check API keys are valid (not expired)
2. Verify service name exists in Datadog
3. Check metric names match your service's metrics

### No report in Lark

**Fix:**
1. Check Lark webhook URL is correct
2. Verify webhook is still active (not deleted)
3. Check GitHub Actions logs for error details

---

## 📝 Example Test Values

**For testing with fprmfdt service:**

```
deployment_time: 2026-05-14T06:00:00Z
commit_sha: <get from: https://github.com/traveloka/fpr-fprmfdt/commits/main>
service_name: fprmfdt
```

**Tips:**
- Use a deployment that happened 1-2 hours ago (so metrics exist)
- Check Datadog first to verify the service has metrics
- Start with wait_minutes=5 for faster testing (default is 15)

---

## 🎯 Success Criteria

Test is successful if:
- ✅ Workflow runs without errors
- ✅ Datadog metrics queried correctly (check JSON artifacts)
- ✅ GitHub Models API generates report
- ✅ Report appears in Lark within 20 minutes
- ✅ Report content makes sense

---

## 📖 Next Steps

After successful self-test:

1. **Integrate with real service:**
   - Copy `examples/caller-workflow.yaml` to service repo
   - Configure secrets in service repo (or org-level)
   - Test with real deployments

2. **Iterate on metrics:**
   - Add more metrics if needed
   - Adjust AI prompt for better analysis
   - Tune wait time

3. **Automate:**
   - Trigger automatically after deployment
   - Integrate with deployment workflow

---

## 🔗 Related Docs

- [Main README](../README.md)
- [Secret Configuration](SECRET-CONFIGURATION.md)
- [PoC Test Guide](POC-TEST-GUIDE.md)

---

**Ready to test? Go to [Actions](https://github.com/jinjinwang11/ai-release-monitoring/actions/workflows/test-self.yaml) and click "Run workflow"!** 🚀
