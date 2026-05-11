# Risk Summary — DCS Care Connect Health 360

## Risk register

Methodology: OWASP Risk Rating × CVSS 3.1 for technical severity, mapped to NIST 800-30 likelihood (qualitative). All risks are tied to a STRIDE category, a MITRE ATT&CK technique where applicable, and a HIPAA Security Rule §164.30x reference.

Ownership: each risk has an accountable team and a target remediation quarter. Compensating controls describe what is in place TODAY while remediation is in flight.

| ID  | Description | STRIDE | MITRE ATT&CK | Component | Severity | Likelihood | Impact | CVSS 3.1 | HIPAA ref | Owner | Target | Compensating control (now) |
|-----|-------------|--------|--------------|-----------|----------|------------|--------|----------|-----------|-------|--------|----------------------------|
| R1  | PHI transmitted to/from clients without enforced TLS 1.2+ on every endpoint (legacy admin panel still allows TLS 1.0) | Information Disclosure | T1040 Network Sniffing | Frontend ↔ Backend, Admin panel | High | Medium | High | 7.4 (High) | §164.312(e)(1) Transmission security | Platform Eng | 2026-Q3 | Admin panel reachable only over corp VPN; certificate pinning planned |
| R2  | Weak authentication: password-only for clinicians, no MFA on admin role, no session re-auth for sensitive actions | Spoofing, Elevation | T1078 Valid Accounts, T1110 Brute Force | IdP, App | High | High | High | 8.1 (High) | §164.312(d) Person/entity authentication | Identity team | 2026-Q3 | Account lockout after 5 failures; password policy 14 chars; SOC alerts on impossible-travel logins |
| R3  | Third-party dependencies not continuously scanned; SBOM not produced per build; supply chain risk (PyPI/npm typosquatting, Log4Shell-class) | Tampering | T1195 Supply Chain | Backend, CI/CD | High | Medium | High | 7.5 (High) | §164.308(a)(1)(ii)(B) Risk management | DevSecOps | 2026-Q3 | Renovate bot + manual quarterly review; runtime egress allowlist on prod nodes |
| R4  | Insufficient audit logging: PHI access events not centralized, no immutable store, retention < 6 years required by HIPAA | Repudiation, Info Disclosure | T1070 Indicator Removal | All | High | Medium | High | 7.1 (High) | §164.312(b) Audit controls; §164.316(b)(2)(i) 6-year retention | SRE + Security | 2026-Q4 | App-level logs to CloudWatch with 90-day retention; gap to 6yr WORM tracked |
| R5  | No tested disaster-recovery / business-continuity plan covering ransomware on PHI database | Denial of Service | T1486 Data Encrypted for Impact | Data plane | Medium | Medium | High | 6.3 (Medium) | §164.308(a)(7) Contingency plan | SRE | 2026-Q4 | Cross-region encrypted snapshots every 6h; RPO 6h / RTO 24h documented but not drilled |
| R6  | API authorization gaps: BOLA / IDOR risk on patient-record endpoints; ABAC not enforced (only RBAC) | Elevation, Info Disclosure | T1213 Data from Information Repositories | Backend API | High | Medium | High | 8.6 (High) | §164.312(a)(1) Access control; §164.502(b) Minimum necessary | Backend team | 2026-Q3 | API gateway scopes per role; SOC alerts on high-volume patient-record reads per user |
| R7  | Phishing risk against staff; no DMARC reject policy; no security-awareness program with metrics | Spoofing | T1566.001 Spear-phishing Attachment | Email / People | High | High | High | n/a (process) | §164.308(a)(5) Security awareness | Security + HR | 2026-Q3 | Email gateway sandboxing; quarterly tabletop |
| R8  | Secrets in environment variables / .env files; no rotation; no central vault | Information Disclosure | T1552.001 Credentials in Files | CI/CD, Backend | Medium | High | High | 7.5 (High) | §164.308(a)(4) Information access management | DevSecOps | 2026-Q3 | Read access restricted to deploy role; quarterly manual rotation |

## Risk rating method

- **Severity** = max(Likelihood, Impact) on the 5×5 OWASP qualitative matrix.
- **Likelihood**: Low (annual), Medium (quarterly), High (monthly+) based on industry incident frequency for healthcare apps (HHS OCR breach portal stats).
- **Impact**: judged against §164.402 breach criteria — number of records, sensitivity (PHI), and probability of misuse.
- **CVSS**: where applicable, scored with environmental metric MA:H (modified availability) and CR:H (confidentiality requirement) given PHI.

## Tracking

Each risk SHOULD be linked to a JIRA epic in the `SEC-DCS` project. Format: `R<n> → SEC-DCS-<ticket>`. Update this table on every quarterly review.

## Out of scope (this iteration)

- Physical safeguards (data center) — handled by AWS Shared Responsibility Model and AWS BAA.
- Workforce training content — owned by HR/Compliance separately, but tracked under R7.
- Medical device integration — DCS Health 360 is web-only in current scope.
