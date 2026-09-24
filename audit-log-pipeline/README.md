# Audit Log Pipeline

A webhook-triggered n8n workflow that captures AI interaction events as tamper-evident audit log entries in Supabase. Designed for organizations that need a queryable, integrity-verified record of all LLM interactions as SOC 2 audit evidence.

**SOC 2 TSC:** CC7.2 (system monitoring), CC9.2 (vendor risk) — supports evidence collection

---

## What it does

1. Receives a POST request containing an AI interaction event
2. Validates that required fields are present (user_id, session_id, model_id, prompt)
3. Generates a unique `audit_id` and a SHA-256 integrity hash of the canonical payload
4. Writes an immutable entry to a Supabase table with a 90-day retention timestamp
5. Returns the `audit_id` and `integrity_hash` to the caller for correlation

Integrity verification: recompute `SHA-256(JSON.stringify({ user_id, session_id, model_id, prompt, response, logged_at }))` against the stored `integrity_hash` at any time to detect modification.

---

## Who it's for

Teams deploying AI systems in SOC 2 environments who need a structured, persistent record of AI interactions. Useful for security engineers building compliance infrastructure and for product teams adding audit capabilities to an existing n8n-based AI workflow.

The same design applies to forensic and investigative workflows where chain of custody for AI interactions is a legal or evidentiary requirement.

**Level:** Intermediate

---

## Chain-of-custody properties

The audit log fields map directly to chain-of-custody requirements:

- **`audit_id`**: unique identifier returned to the caller; functions as an evidence item reference for correlation and retrieval
- **`integrity_hash`**: SHA-256 of the canonical payload; recompute at any time to verify the record has not been modified since it was logged
- **Append-only INSERT**: no application-layer UPDATE or DELETE; the entry is immutable after creation
- **`logged_at`** + **`user_id`** + **`session_id`**: establish who interacted with which AI model, in which session, at what time

For AI-assisted investigation workflows, these properties establish a chain of custody for every AI interaction that contributed to a decision or finding. See [Chain of Custody](https://github.com/nenedesign/ai-accountability-design-patterns/blob/main/concepts/05-chain-of-custody.md) in ai-accountability-design-patterns for the design framework.

---

## Nodes used

- **Webhook** — receives POST requests on `/soc2-audit-log` with API key auth
- **Set** — normalizes fields from the request body
- **Code** — validates required fields, generates audit_id (SHA-256) and retention timestamp
- **IF** — routes valid entries to Supabase and invalid entries to a 400 response
- **HTTP Request** — append-only INSERT to Supabase REST API
- **Respond to Webhook (x2)** — returns 200 with audit_id or 400 with error

---

## Requirements

**Services:**
- n8n (self-hosted or cloud)
- Supabase project with a `ai_audit_log` table (schema below)

**Credentials:**
- A header auth credential for the webhook API key
- Supabase anon key (for `apikey` header)
- Supabase service key (for `Authorization` header, needed to bypass RLS on INSERT)

---

## Supabase table setup

Run this SQL in your Supabase SQL editor before activating the workflow:

```sql
create table ai_audit_log (
  id bigint generated always as identity primary key,
  audit_id text not null unique,
  user_id text not null,
  session_id text not null,
  model_id text not null,
  prompt text not null,
  response text,
  logged_at timestamptz not null,
  integrity_hash text not null,
  retention_expires_at timestamptz not null,
  status text not null default 'active' check (status in ('active', 'expired')),
  created_at timestamptz default now()
);

-- Append-only: application layer can INSERT but not UPDATE or DELETE
create policy "insert_only" on ai_audit_log for insert using (true);
alter table ai_audit_log enable row level security;
```

---

## How to import

1. Download [workflow.json](workflow.json)
2. Open your n8n instance and click **+** → **Import from file** → select the file
3. Run the Supabase SQL above to create the table and RLS policy
4. Replace `YOUR_SUPABASE_PROJECT_URL`, `YOUR_SUPABASE_ANON_KEY`, and `YOUR_SUPABASE_SERVICE_KEY` with your values in the **Write to Supabase** node
5. Add a header auth credential to the **Receive Audit Event** webhook node
6. Activate the workflow

---

## Example request

```bash
curl -X POST https://your-n8n-instance/webhook/soc2-audit-log \
  -H "Authorization: Bearer YOUR_WEBHOOK_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "usr_abc123",
    "session_id": "sess_xyz789",
    "model_id": "claude-sonnet-4-6",
    "prompt": "Summarize the Q3 earnings report.",
    "response": "Q3 revenue increased 12% YoY...",
    "logged_at": "2026-09-07T14:30:00.000Z"
  }'
```

**Example response (200):**
```json
{
  "audit_id": "aud_a3f9c2b1d4e7",
  "status": "logged",
  "integrity_hash": "e3b0c44298fc1c149afb..."
}
```

**Example response (400):**
```json
{
  "error": "Missing required fields: prompt"
}
```

---

## Customization

- **Retention period:** Change `90 * 24 * 60 * 60 * 1000` in the Code node to adjust the retention window (value is in milliseconds)
- **Required fields:** Add or remove fields from the `missing.push()` validation block in the Code node
- **PII handling:** If prompts contain PII, run the [LLM02 PII Detector](https://github.com/nenedesign/ai-governance-owasp10/tree/main/workflows/llm02-pii-detector) before this workflow to redact sensitive content before it enters the audit log
- **Alerting:** Wire the Supabase response into a Slack node to notify on high-volume logging or errors

---

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/)
