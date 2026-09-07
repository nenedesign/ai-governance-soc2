# SOC 2: Trust Service Criteria Reference

**Standard:** SOC 2 (Service Organization Control 2)  
**Framework:** Trust Service Criteria (TSC), published by the American Institute of Certified Public Accountants (AICPA)  
**Current version:** 2017 Trust Service Criteria (as revised)  
**Official source:** https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services

---

## Why this file exists

The PCI-DSS reference in [ai-governance-pci-dss](https://github.com/nenedesign/ai-governance-pci-dss) explains why that standard is not pinned: PCI SSC holds copyright and restricts redistribution.

SOC 2 carries the same restriction. The Trust Service Criteria document is copyrighted by the AICPA. It is available through AICPA membership or purchase. This file does not reproduce TSC text. It describes the specific criteria this repository's artifacts are built against, in original language, and links to the authoritative source.

---

## Trust Service Criteria structure

SOC 2 is organized into five trust service categories. Security (Common Criteria, "CC") is required in all SOC 2 engagements. The others are optional and in scope only if the organization includes them.

| Category | Prefix | Required? |
|----------|--------|-----------|
| Security | CC | Yes |
| Availability | A | Optional |
| Processing Integrity | PI | Optional |
| Confidentiality | C | Optional |
| Privacy | P | Optional |

The artifacts in this repository address criteria in the Security category (CC) and, if Processing Integrity is in scope, PI1.

---

## Criteria in scope

### CC7.2: System Operations Monitoring

**What it requires:** The entity monitors system components and the operation of those controls on an ongoing basis to detect anomalies and address threats that may indicate failures of controls. This includes monitoring for unauthorized access, changes to configurations, and events that could affect the entity's ability to meet its commitments.

**Why it applies to LLM deployments:** An LLM system that processes user inputs and returns outputs is a system component subject to CC7.2. Without a persistent, queryable record of interactions, the monitoring requirement cannot be met. Auditors reviewing CC7.2 need evidence that interactions were captured, that the capture mechanism was operating, and that the records are intact.

**How the Audit Log Pipeline supports evidence collection for CC7.2:**
- Every AI interaction is captured as a structured row in Supabase before the response is returned
- The SHA-256 integrity hash allows a reviewer to recompute the hash from the stored fields and detect any modification after the fact
- The `audit_id` returned to the caller allows correlation between application logs and audit log entries
- The audit log is append-only at the application layer; the pipeline can only INSERT, never UPDATE or DELETE

**What CC7.2 requires beyond this artifact:** Actual monitoring means someone (or something) reviews the log for anomalies. The pipeline creates the evidence; CC7.2 requires a process to act on it — alerting, regular review, or automated anomaly detection. Neither artifact in this repository implements that review process.

---

### CC9.2: Risk Mitigation (Vendor and Business Partner Risk)

**What it requires:** The entity assesses and manages risks associated with vendors and business partners. This includes evaluating vendor controls, monitoring vendor performance, and maintaining an understanding of the vendor's role in the entity's system.

**Why it applies to LLM deployments:** When an organization uses a third-party model provider (OpenAI, Anthropic, Google, a self-hosted open-weight model), that provider is a vendor. CC9.2 asks: how do you know what you're sending them, how often, and under what conditions? An audit log that captures `model_id`, `prompt`, and `logged_at` per interaction provides a foundation for vendor risk review.

**How the Audit Log Pipeline supports evidence collection for CC9.2:**
- `model_id` documents which vendor model received each interaction
- If the model changes (vendor migration, version upgrade), the log captures the transition point
- Prompt logging enables periodic review of what data was sent to a vendor's API

**What CC9.2 requires beyond this artifact:** Vendor risk management requires documented risk assessments, contractual controls (DPA, data processing agreements), and periodic review of vendor security posture. The pipeline supports evidence collection but does not perform risk assessment.

---

### CC6.5: Logical and Physical Access Controls (Discontinuation)

**What it requires:** The entity discontinues logical access to protected information assets when the access is no longer needed. This includes removing access when employees leave, revoking access tied to system changes, and managing the lifecycle of credentials.

**Why it applies to audit log retention:** Audit log entries containing user prompts and AI responses are protected information. Retaining them indefinitely creates unnecessary long-term exposure: past entries accumulate, access scope grows, and data that should have aged out remains queryable. A retention lifecycle that flags entries as expired demonstrates that the organization manages the lifecycle of audit data rather than retaining it without bound.

**How the Retention Enforcer supports evidence collection for CC6.5:**
- Entries are flagged `expired` after 90 days, creating a documented retention boundary
- The enforcer runs daily and logs its output (`entries_expired`, `enforced_at`), producing evidence of consistent enforcement
- Expired entries remain in the table (they are not deleted), which supports audit trail continuity while limiting the active working set

**Important:** `expired` status is a flag applied by the Retention Enforcer at the application layer. To prevent the application from reading expired entries, add a Supabase RLS policy that excludes `status = 'expired'` rows from SELECT queries for non-administrative roles. The RLS policy is the actual access control; the flag is the mechanism that triggers it.

---

### CC2.2 and CC3.1: Communication and Risk Assessment

**CC2.2 (Internal Communication):** The entity internally communicates information necessary to support the functioning of internal controls. For AI systems, this means: does the organization have documented, shared understanding of what the AI system does, what data it processes, what its access controls are, and what risks it carries?

**CC3.1 (Risk Assessment Objectives):** The entity specifies objectives with sufficient clarity to enable the identification and assessment of risks relating to objectives.

**How the SOC 2 AI Evidence Template supports evidence collection for CC2.2 and CC3.1:**
- The system description section documents what the AI system does and who operates it
- The TSC mapping section formalizes which criteria apply and how controls address them
- The access control log documents who has access to what at the AI system layer
- The incident record captures AI-specific events that may not appear in standard incident logs
- The model change log tracks when the underlying model changes, which is a risk event

---

## Artifact-to-TSC mapping

| Artifact | CC7.2 | CC9.2 | CC6.5 | CC2.2 | CC3.1 |
|----------|-------|-------|-------|-------|-------|
| [Audit Log Pipeline](../audit-log-pipeline/workflow.json) | Provides tamper-evident interaction log for monitoring | Captures model_id per interaction for vendor review | N/A | N/A | N/A |
| [Retention Enforcer](../retention-enforcer/workflow.json) | N/A | N/A | Enforces 90-day lifecycle on audit log entries | N/A | N/A |
| [SOC 2 AI Evidence Template](../soc2-ai-evidence-template.md) | N/A | N/A | N/A | Structures internal documentation of AI system controls | Frames risk assessment objectives for AI systems in scope |

---

## Compliance note

These artifacts support SOC 2 evidence collection. They do not constitute SOC 2 compliance and cannot be used as a substitute for a formal SOC 2 Type II audit by a licensed CPA firm with SOC 2 attestation capability.

SOC 2 Type II requires:
- A defined system description and trust services criteria scope
- Controls that operated effectively over the audit period (typically 6 or 12 months)
- Independent testing by a qualified auditor
- A formal attestation report issued by the auditing firm

These artifacts produce evidence that supports an audit. Whether that evidence is sufficient depends on the scope of the engagement, the auditor's testing approach, and the controls the organization has in place beyond these workflows.

---

## Reference

American Institute of Certified Public Accountants (AICPA). *Trust Service Criteria for Security, Availability, Processing Integrity, Confidentiality, and Privacy.* Available at: https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services
