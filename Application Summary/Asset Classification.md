# Asset Classification & Criticality — DCS Care Connect Health 360

Classifying every data element and system component lets us decide *which* controls are mandatory vs. nice-to-have and *which* breaches trigger the HIPAA Breach Notification Rule.

## Data classification scheme

| Tier | Label | Examples | Handling |
|------|-------|----------|----------|
| 1 | **Restricted — PHI / PII** | Patient name, MRN, DOB, diagnoses, lab results, images, SSN | Encrypted at rest + in transit; access logged per-record; minimum-necessary; HIPAA Breach Notification triggered on unauthorized disclosure |
| 2 | **Confidential** | Clinician credentials, internal pricing, BAA contracts, API keys, source code | Encrypted at rest + in transit; access on least-privilege; loss is material but not a Privacy Rule breach |
| 3 | **Internal** | Application logs without PHI, dashboards, runbooks | Access for employees; encryption in transit required |
| 4 | **Public** | Marketing site, public documentation | No special handling |

**Rule of thumb**: if a single record in a column can identify a patient *or* reveal a health condition, the entire dataset is Tier 1. Pseudonymized data is still Tier 1 unless it meets HIPAA Safe Harbor §164.514(b)(2) (18 identifiers removed) or Expert Determination (b)(1).

## Patient / user criticality tiers

| Tier | Population | Why elevated | Extra controls |
|------|-----------|--------------|----------------|
| VIP | Public figures, employees, minors, behavioral-health patients | Heightened reputational & legal risk on disclosure | Break-glass-only access; per-record audit; dual approval for export |
| Standard | All other patients | HIPAA baseline | RBAC + audit logging |
| Test | Synthetic / de-identified | Used in non-prod | Verification that no real PHI present (CI check) |

## Component criticality (BIA — Business Impact Analysis)

| Component | Criticality | RPO | RTO | Notes |
|-----------|-------------|-----|-----|-------|
| PHI Database (primary) | Tier 1 (mission-critical) | 6 h | 24 h | Patient-care impact within 1 hour; cross-region replicas required |
| Audit Log Store | Tier 1 | 1 h | 4 h | HIPAA §164.312(b); loss = audit gap |
| API Gateway | Tier 1 | n/a | 1 h | Stateless; recoverable from IaC |
| Identity Provider | Tier 1 | n/a | 4 h | Outage blocks all access; standby tenant in alt region |
| Object Storage (PHI attachments) | Tier 1 | 24 h | 24 h | Versioning + replication enabled |
| Email service | Tier 2 | 4 h | 8 h | Outage delays notifications, not care |
| Analytics (de-identified) | Tier 3 | 7 d | 7 d | No PHI; rebuild from source |

## Asset inventory (illustrative — keep in sync with CMDB)

| Asset ID | Description | Tier | Owner | Last review |
|----------|-------------|------|-------|-------------|
| A-001 | PHI Database — `dcs-prod-rds-postgres` | Restricted | DBA team | 2026-04 |
| A-002 | Patient attachments — `s3://dcs-phi-attachments-prod` | Restricted | Backend team | 2026-04 |
| A-003 | Audit log store — `s3://dcs-audit-worm` (Object Lock) | Restricted | Security | 2026-04 |
| A-004 | Identity provider — Okta tenant `dcs.okta.com` | Confidential | Identity team | 2026-04 |
| A-005 | CI/CD secrets — Vault path `secret/dcs/ci/*` | Confidential | DevSecOps | 2026-04 |
| A-006 | API Gateway config — IaC repo `infra/api-gw` | Confidential | Platform | 2026-04 |

## Lifecycle obligations per tier

- **Tier 1 retention**: PHI per state law minimum (often 7–10 years for adults, 21+ for minors). HIPAA Security Rule §164.316(b)(2) requires documentation retention 6 years separately.
- **Destruction**: Tier 1 data must be cryptographically erased (key destruction) or physically destroyed per NIST SP 800-88. A certificate of destruction is retained 6 years.
- **De-identification**: any data leaving the Restricted tier must pass Safe Harbor or Expert Determination, documented per dataset.

## Mapping into the threat model

Asset criticality feeds:
- **Impact column** in `RISK SUMMARY.md` — Tier 1 asset compromise = High Impact by default.
- **STRIDE prioritization** — controls for Tier 1 components are mandatory, not advisory.
- **Runbook severity** — `Attack Scenario {1..4}/Runbook.md` set MTTD/MTTR targets based on tier.
