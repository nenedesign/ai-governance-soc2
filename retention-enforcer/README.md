# Retention Enforcer

A daily scheduled n8n workflow that enforces the 90-day retention policy on the SOC 2 audit log table. Runs at 2:00 AM, flags all active entries past their `retention_expires_at` date as `expired`, and returns a count of entries updated per run.

**SOC 2 TSC:** CC6.5 (logical access controls: discontinuation) — supports evidence collection

---

## What it does

1. Fires at 2:00 AM on a daily cron schedule
2. Sends a PATCH request to the Supabase REST API filtering on `status=eq.active` and `retention_expires_at=lt.{now}`
3. Updates all matching entries from `status: active` to `status: expired` in a single batch call
4. Returns a count of entries expired and the enforcement timestamp

**Important:** Entries are flagged `expired`, not deleted. The audit log is append-only; entries must never be removed because an audit trail requires complete history. To limit access to expired entries, configure a Supabase RLS SELECT policy that excludes `status = 'expired'` rows for non-administrative roles.

---

## Who it's for

Security engineers and DevOps teams who have deployed the [Audit Log Pipeline](../audit-log-pipeline/README.md) and need automated retention lifecycle management. The Retention Enforcer runs without manual intervention and produces a daily enforcement record that can be included in SOC 2 evidence.

**Level:** Beginner

---

## Nodes used

- **Schedule Trigger** — fires daily at 2:00 AM via cron (`0 2 * * *`)
- **HTTP Request** — PATCH to Supabase REST API with server-side filter
- **Code** — counts updated entries and formats the enforcement log entry

---

## Requirements

**Services:**
- n8n (self-hosted or cloud)
- Supabase project with the `ai_audit_log` table from the [Audit Log Pipeline](../audit-log-pipeline/README.md)

**Credentials:**
- Supabase anon key (for `apikey` header)
- Supabase service key (for `Authorization` header)

---

## How to import

1. Download [workflow.json](workflow.json)
2. Open your n8n instance and click **+** → **Import from file** → select the file
3. Replace `YOUR_SUPABASE_PROJECT_URL`, `YOUR_SUPABASE_ANON_KEY`, and `YOUR_SUPABASE_SERVICE_KEY` with your values in the **Flag Expired Entries** node
4. Activate the workflow

The workflow will run at 2:00 AM in the timezone configured in your n8n instance. Adjust the cron expression if a different time is preferred.

---

## Customization

- **Retention window:** The 90-day window is set in the Audit Log Pipeline (the `retention_expires_at` field). The Retention Enforcer enforces whatever value is in that field — no change needed here if you adjust the window in the pipeline
- **Schedule time:** Change `0 2 * * *` in the Schedule Trigger to any valid cron expression
- **Deletion:** To permanently delete expired entries instead of flagging them, change `{"status": "expired"}` in the PATCH body to a DELETE HTTP method. Only do this if your organization's policy permits deletion of audit records and your legal/compliance team has approved it
- **Alerting:** Wire the Count node output into a Slack node to receive a daily enforcement summary

---

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/)
