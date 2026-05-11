# STRIDE Analysis per Component — DCS Care Connect Health 360

STRIDE applied to each component of the DFD (`Data Flow Diagram.md`). For each (Component × STRIDE category) we list the threat, its likelihood and impact, the MITRE ATT&CK technique (where applicable), and the existing / proposed control.

Legend: **S** Spoofing · **T** Tampering · **R** Repudiation · **I** Information Disclosure · **D** Denial of Service · **E** Elevation of Privilege.

## Frontend (React, EC2/ECS)

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Session-token theft via stored XSS — attacker impersonates clinician | T1539 Steal Web Session Cookie | M/H | Strict CSP (`default-src 'self'`); HttpOnly + Secure + SameSite=Strict cookies; subresource integrity for vendor scripts |
| T | Malicious browser extension or MITM rewrites form data | T1557 Adversary-in-the-Middle | L/H | TLS 1.3 + HSTS preload; CSP `connect-src` allowlist; integrity hashes on bundle |
| R | User denies submitting clinical data | T1070 | L/M | Signed JWT + server-side audit log linked to userId hash + IP + UA |
| I | Sensitive data cached in browser local storage | — | M/H | Forbid PHI in localStorage / sessionStorage; in-memory only; auto-clear on visibility change |
| D | UI overload via huge payload from API or client-side rendering bomb | T1499 | L/M | Pagination, virtualized lists, payload size caps server-side |
| E | XSS → run JS as authenticated user | T1059.007 Client-side Scripting | M/H | Output encoding via framework; CSP nonces; DOMPurify on rich-text fields; security-headers test in CI |

## API Gateway

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Forged JWT (weak alg, leaked signing key) | T1606.001 Forge Web Credential | M/H | Asymmetric RS256 / EdDSA; `alg` allowlist; key rotation 90d via JWKS |
| T | Mass-assignment on PATCH endpoints overrides role | T1550 Use Alternate Auth Material | M/H | Strict schema validation per endpoint; deny unknown fields |
| R | Lack of request-level audit for non-PHI endpoints | — | M/M | Structured access log with request ID propagated downstream |
| I | Verbose error responses leak stack traces / internals | T1213 | M/M | Generic error envelope; debug info only in non-prod |
| D | API flooding / Layer-7 DDoS | T1498.001 | M/H | WAF + rate-limit per token & per IP; circuit breaker to backends |
| E | BOLA — patient A accesses patient B's record by changing ID | T1213 | H/H | Authorization at the object level: server enforces `record.ownerId == subject.id OR clinician∈care-team` |

## Identity Provider (Cognito / Okta)

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Credential stuffing using breach corpora | T1110.004 Credential Stuffing | H/H | MFA required for all staff; password-breach check on rotate; impossible-travel detection |
| T | Account-recovery flow bypass | T1078 | M/H | Recovery via verified second factor only; risk-based step-up |
| R | Disabled audit on admin role mutations | — | L/H | Tamper-evident admin event log shipped to immutable store |
| I | OIDC discovery exposes internal claims | — | L/M | Trim claims to minimum-necessary; no PHI inside ID token |
| D | Lockout abuse to DoS legit users | T1531 | M/M | Lockout on identity+IP combination, not identity alone |
| E | Privilege escalation via group membership injection | T1078.004 | M/H | Groups managed via SCIM only; manual changes flagged in SIEM |

## Backend Services

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Service-to-service spoofing inside VPC | T1550 | M/H | mTLS between services; SPIFFE/SPIRE for workload identity |
| T | SQL injection on search endpoints | T1190 | M/H | Parameterized queries enforced; static analysis in CI; WAF rule for SQLi |
| T | Deserialization of untrusted input | T1190 | L/H | No native object deserialization on user input; JSON only with schema validation |
| R | Background jobs mutate records without operator attribution | — | M/M | Every mutation tagged with `actor=service:<id>` and `correlation_id`; reviewed weekly |
| I | Logs contain PHI fields | T1530 | M/H | Logging library masks fields by schema (SSN, DOB, MRN); CI test for log fixtures |
| D | Unbounded query / slow regex (ReDoS) | T1499.004 | M/M | Query timeout; safe regex linter; circuit breakers |
| E | Container escape from compromised service | T1611 | L/H | Read-only root FS; non-root user; seccomp / AppArmor profile; image signing |

## PHI Database

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Backup restored into untrusted environment | T1078 | L/H | Encrypted snapshots with separate KMS key; restore requires break-glass approval |
| T | Direct DB write bypassing app (DBA with prod access) | T1565 | L/H | No human DB access in prod; just-in-time access via session-recorded bastion |
| R | Schema migrations without change record | — | M/M | All migrations versioned in git; applied via pipeline only |
| I | Data exfil via legitimate read by compromised app account | T1213 | M/H | Row-level access logs; anomaly detection on bulk-read patterns |
| D | Ransomware encrypts DB volume | T1486 | M/H | Cross-region snapshots; air-gapped backup with 7-day immutability |
| E | SQL function with `SECURITY DEFINER` running as superuser | T1068 | L/H | Function review in PR; least-privilege role for DB connections |

## Object Storage (PHI attachments)

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Pre-signed URL replay by attacker on shoulder | T1550 | M/H | Short TTL (≤ 5 min); bind URL to source IP / user agent when feasible |
| T | Bucket misconfig — public ACL | T1530 | M/H | Block-public-access at account level; Config rule alerts on drift |
| R | No object-level access log | — | M/M | S3 server-access + CloudTrail data events enabled, shipped to immutable log |
| I | Cross-tenant access via path traversal | T1530 | M/H | Per-tenant prefix + IAM condition `s3:prefix` |
| D | Cost-DOS via excessive uploads | T1499.003 | L/M | Per-user upload quota; pre-signed URL count cap |
| E | KMS key policy allows unintended principal | T1078.004 | L/H | KMS key policy reviewed in IaC PRs; CloudTrail alarms on policy change |

## Audit Log Store (WORM)

| STRIDE | Threat | MITRE | L/I | Control |
|--------|--------|-------|-----|---------|
| S | Forged log entries injected by compromised app | T1070.001 | L/H | Logs include HMAC chain; tampering detected at read time |
| T | Retention shortened to hide activity | T1070 | L/H | Object Lock COMPLIANCE mode for 6 years; deletion requires AWS account-level break-glass with two-person approval |
| I | Logs queried at scale to enumerate users | — | L/M | Query access role separate from app role; logged itself |

## Cross-cutting controls

- **Secrets**: all in vault (AWS Secrets Manager / Vault), rotated ≤ 90 days, never in env files or git.
- **CI/CD**: SBOM per build; dependency review on PR; container image scanned; code signed.
- **Observability**: trace IDs propagated end-to-end; PHI fields hashed in traces.
- **Detection**: SIEM rules per MITRE technique used above; on-call paged for high-confidence detections.

## Gaps tracker

Each row above whose "Control" is described in future tense (e.g., "WebAuthn for admins — rolling out") is a tracked risk in `RISK SUMMARY.md`. Quarterly, walk this matrix and re-classify control as "in place" or "open".
