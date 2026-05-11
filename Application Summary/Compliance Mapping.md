# HIPAA & Compliance Mapping — DCS Care Connect Health 360

DCS Care Connect Health 360 handles Protected Health Information (PHI) and is therefore subject to the HIPAA Security Rule (45 CFR Part 164, Subpart C). This document maps the threat-model risks (R1–R8) to the specific safeguards each HIPAA section requires.

It is not legal advice. It is the technical control matrix that engineering and security use to demonstrate compliance during audits and to triage findings.

## Applicable regulations

| Regulation | Why it applies | Owner |
|-----------|----------------|-------|
| HIPAA Security Rule (45 CFR 164.302–318) | PHI is created, received, maintained, transmitted | Security, Legal |
| HIPAA Privacy Rule (45 CFR 164.500–534) | Minimum-necessary, patient access rights, accounting of disclosures | Privacy Office |
| HIPAA Breach Notification Rule (45 CFR 164.400–414) | 60-day notification on PHI breach | Security + Legal + Comms |
| HITECH Act (2009) | Tiered civil penalties up to $1.5M / violation category / year | Legal |
| State law (e.g., CCPA / CMIA in California, Texas HB300) | May add stricter notification or data-rights timelines | Legal |
| SOC 2 Type II | Customer / partner contractual requirement | Security |
| PCI DSS | Only if payment processing — currently delegated to BAA processor (out of scope of this model) | Finance + Security |

## Safeguard mapping (Security Rule)

### Administrative safeguards (45 CFR 164.308)

| HIPAA citation | Requirement | Risks addressed | Control evidence |
|----------------|-------------|------------------|------------------|
| §164.308(a)(1)(ii)(A) | Risk analysis | R1–R8 | This threat model + annual review |
| §164.308(a)(1)(ii)(B) | Risk management | R3 | SDLC requires risk acceptance / mitigation per finding |
| §164.308(a)(3) | Workforce security (authorization, supervision, termination) | R2, R6 | Joiner-Mover-Leaver runbook; IAM lifecycle automation |
| §164.308(a)(4) | Information access management | R6, R8 | RBAC matrix; least-privilege review quarterly; secrets in vault |
| §164.308(a)(5) | Security awareness and training | R7 | Quarterly phishing simulation + training; metrics tracked |
| §164.308(a)(6) | Security incident procedures | R4, R5 | IR playbook; on-call rotation; tabletop quarterly |
| §164.308(a)(7) | Contingency plan | R5 | DR plan with RPO 6h / RTO 24h; tested annually |
| §164.308(a)(8) | Evaluation | all | Annual penetration test; SOC 2 audit |

### Physical safeguards (45 CFR 164.310)

Out of scope for application threat model — covered by AWS BAA. Verify annually that:
- AWS BAA is current and lists every region/service in use.
- No PHI is processed on non-BAA-covered services (e.g., consumer Slack/Notion).

### Technical safeguards (45 CFR 164.312)

| HIPAA citation | Requirement | Risks addressed | Control evidence |
|----------------|-------------|------------------|------------------|
| §164.312(a)(1) | Access control (unique user ID, emergency access, automatic logoff, encryption/decryption) | R2, R6 | OIDC IdP; session timeout 15 min; KMS-managed keys |
| §164.312(b) | Audit controls | R4 | Immutable WORM log of PHI access; SIEM ingestion |
| §164.312(c)(1) | Integrity controls | R3 | SBOM, code signing, DB row-level hashes for clinical records |
| §164.312(d) | Person or entity authentication | R2 | MFA for staff (rolling out); WebAuthn for admins |
| §164.312(e)(1) | Transmission security | R1 | TLS 1.2+ enforced; HSTS; certificate transparency monitoring |

### Organizational / Policy (45 CFR 164.314, 164.316)

| HIPAA citation | Requirement | Evidence |
|----------------|-------------|----------|
| §164.314(a) | Business Associate contracts | BAA on file for every vendor that touches PHI |
| §164.316(a) | Policies and procedures | Security policy reviewed annually; signed by CISO |
| §164.316(b)(2)(i) | Retain documentation 6 years | All audit logs, risk assessments retained ≥ 6y |

## Breach Notification readiness (45 CFR 164.400)

Trigger criteria (any one):
1. Unsecured PHI acquired, accessed, used or disclosed in a way that violates the Privacy Rule.
2. Risk assessment per §164.402(2) does NOT conclude low probability of compromise across four factors:
   - Nature & extent of PHI involved
   - Unauthorized person who used / received
   - Whether PHI was actually acquired or viewed
   - Mitigation extent

Notification timeline:
- **< 60 days** from discovery — written notice to individuals.
- **< 60 days** — HHS Secretary (immediately if > 500 individuals; annually otherwise).
- **< 60 days** — Prominent media notice if > 500 individuals in same state.

Internal SLA:
- T+0 (discovery): Incident commander assigned, evidence preserved.
- T+1 day: Forensic scope + count of impacted records.
- T+5 days: §164.402 risk assessment complete.
- T+10 days: Draft notifications reviewed by Legal.
- T+30 days: Notifications dispatched (well before 60-day legal deadline).

## Audit-evidence index

Maintain pointers to evidence (not the artifacts themselves) here:

- Risk register → `RISK SUMMARY.md` (this repo)
- DFD with trust boundaries → `Application Summary/Data Flow Diagram.md`
- STRIDE per-component → `Application Summary/STRIDE Analysis.md`
- Asset classification → `Application Summary/Asset Classification.md`
- Attack scenarios + runbooks → `Attack Scenario {1..4}/`
- Penetration test report → `${EVIDENCE_VAULT}/pentest/${year}.pdf`
- BAA registry → `${EVIDENCE_VAULT}/baa/`
- Training completion records → HRIS export, quarterly

## State-specific addenda (track only if applicable)

- **California (CMIA, CCPA/CPRA)**: 15-day breach notification under CMIA, broader civil-action standing.
- **Texas (HB 300)**: Stricter than HIPAA on training and breach notice (60 days but to AG too).
- **EU residents (GDPR)**: 72-hour authority notification; separate from HIPAA.

If DCS serves patients in any of the above, the IR playbook must include the state-specific track.
