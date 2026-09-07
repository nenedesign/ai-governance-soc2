# SOC 2 AI System Evidence Template

**Purpose:** Use this template to document an AI system that is in scope for a SOC 2 Type II audit. One completed template per AI system. Keep it version-controlled and update it when the system, its controls, or the underlying model changes.

This is not a compliance certification. It is a structured documentation artifact that supports auditor review of AI-specific controls. Fill it in before your audit period begins and treat it as a living document.

---

## 1. System Identification

| Field | Value |
|-------|-------|
| System name | |
| System owner | |
| Date first deployed | |
| Date this document last updated | |
| Current version / build | |
| Environment (production / staging / dev) | |

---

## 2. System Description

**What this system does** (2-4 sentences: inputs, outputs, downstream effects):

> _Example: This system receives customer support queries via a webhook, routes them to Claude claude-sonnet-4-6 for classification and response generation, and posts the response to a Slack channel. It has access to a Supabase knowledge base via embedding search. It does not have write access to any customer data systems._

**System description:**


---

**Data processed by this system:**

| Data type | Classification | Retained? | Retention period |
|-----------|---------------|-----------|-----------------|
| User prompts | | | |
| AI responses | | | |
| Session identifiers | | | |
| User identifiers | | | |
| Other: | | | |

---

## 3. Trust Service Criteria Mapping

Mark the criteria that apply to this system and describe how each is addressed. Leave rows blank if a criterion is not in scope for this engagement.

| Criterion | In scope? | How addressed |
|-----------|-----------|---------------|
| CC7.2: System monitoring | | |
| CC9.2: Vendor/business partner risk | | |
| CC6.1: Logical access controls | | |
| CC6.5: Access discontinuation | | |
| CC2.2: Internal communication | | |
| CC3.1: Risk assessment objectives | | |
| PI1.2: Complete and accurate processing | | |
| Other: | | |

---

## 4. AI Model and Vendor Details

| Field | Value |
|-------|-------|
| Model provider | |
| Model name / version | |
| API endpoint used | |
| Data processing agreement in place? (Y/N) | |
| Zero data retention agreed? (Y/N) | |
| Vendor security documentation reviewed? | |
| Date of last vendor risk review | |

**Known data handling commitments from the vendor:**


---

## 5. Access Control Log

Document who has access to this system and at what level. Update when access changes.

| Name / Role | Access type | Granted date | Revoked date |
|-------------|-------------|--------------|--------------|
| | | | |
| | | | |

**API key / credential management:**

| Credential | Where stored | Rotation frequency | Last rotated |
|------------|-------------|-------------------|--------------|
| Webhook API key | | | |
| Supabase service key | | | |
| LLM API key | | | |

---

## 6. Audit Log Coverage

Confirm that audit logging is in place and describe the coverage.

| Field | Value |
|-------|-------|
| Audit log workflow in place? (Y/N) | |
| Log destination | |
| Fields captured | |
| Integrity mechanism | |
| Retention period enforced | |
| Retention enforcer in place? (Y/N) | |

**Audit log gap analysis** (interactions or events not currently captured):


---

## 7. Incident Record

Log AI-specific incidents that may not appear in standard incident management systems. Include: unexpected model outputs, PII exposure, prompt injection attempts, model degradation events, and any interaction flagged by a human reviewer.

| Date | Description | Severity | Resolution | Reviewer |
|------|-------------|----------|------------|---------|
| | | | | |
| | | | | |

---

## 8. Model Change Log

Track changes to the underlying model or system prompt. A model version change is a risk event: behavior, capabilities, and data handling terms may all change.

| Date | Change | Previous value | New value | Approved by |
|------|--------|----------------|-----------|-------------|
| | Model version update | | | |
| | System prompt change | | | |

---

## 9. Audit Review Schedule

| Activity | Frequency | Owner | Last completed |
|----------|-----------|-------|----------------|
| Review audit log for anomalies | | | |
| Review access control list | | | |
| Vendor risk review | | | |
| Incident log review | | | |
| Model change log review | | | |
| Evidence template update | | | |

---

## 10. Attestation

By signing below, the system owner confirms that this document accurately describes the AI system, its controls, and the current state of evidence collection as of the date indicated.

| Field | Value |
|-------|-------|
| System owner name | |
| Title | |
| Date | |
| Signature | |

---

*This template is provided under MIT License. It is not a legal document and does not constitute SOC 2 compliance. Adapt it to the specific requirements of your audit engagement and auditor.*
