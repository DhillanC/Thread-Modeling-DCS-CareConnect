# Attack Scenario 2 — Compromise of machine processes & cloud data lake

## Threat summary

An attacker compromises a CI/CD service account or a cloud-workload identity (instance profile, container task role) and pivots into the analytics data lake where de-identified-but-rich healthcare data lives. The goal is bulk data theft and / or persistence inside the cloud account.

Why this scenario matters: data-lake credentials are often broader than app DB credentials, run on long-lived workloads, and are watched less aggressively than user accounts.

## Kill-chain mapping

| Phase | Technique | Action |
|-------|-----------|--------|
| Recon | T1593 Search Open Websites | Find public IaC repo or open S3 bucket via search engines |
| Resource Dev | T1583.006 Acquire Infrastructure: Web Services | Stand up exfil S3 in attacker AWS account |
| Initial Access | T1078.004 Valid Accounts: Cloud Accounts | Use leaked CI token / IAM key |
| Execution | T1059.009 Cloud API | aws-cli / boto3 from attacker workstation |
| Persistence | T1098.003 Account Manipulation: Additional Cloud Roles | Attach broad policy to compromised role; create access key |
| Privilege Escalation | T1078.004 / T1098.001 | iam:PassRole abuse, role assumption chain |
| Defense Evasion | T1562.008 Disable Cloud Logs | Stop CloudTrail trail or change S3 bucket logging |
| Discovery | T1580 Cloud Infrastructure Discovery | List buckets, tables, secrets, KMS keys |
| Lateral Movement | T1021.007 Cloud Services | Hop into analytics warehouse, glue jobs, EMR |
| Collection | T1530 Data from Cloud Storage Object | s3:GetObject across PHI prefixes |
| C2 | T1102 Web Service | Egress to attacker-owned S3 / blob over HTTPS |
| Exfiltration | T1567.002 Exfiltration to Cloud Storage | `aws s3 cp --recursive` to attacker bucket |
| Impact | T1485 Data Destruction | Delete CloudTrail / logs to cover tracks |

## Diagram

```mermaid
flowchart LR
    style Recon fill:#0037b8,stroke:#000,stroke-width:2px,color:#fff
    style InitialAccess fill:#F5B041,stroke:#000,stroke-width:2px
    style PrivEsc fill:#EB984E,stroke:#000,stroke-width:2px
    style DefenseEvasion fill:#E59866,stroke:#000,stroke-width:2px
    style Discovery fill:#DC7633,stroke:#000,stroke-width:2px
    style Collection fill:#CA6F1E,stroke:#000,stroke-width:2px,color:#fff
    style Exfiltration fill:#BA4A00,stroke:#000,stroke-width:2px,color:#fff
    style Impact fill:#8a0111,stroke:#000,stroke-width:2px,color:#fff

    Recon[Recon: leaked IaC repo, misconfigured S3] -->|Token or key found| InitialAccess[Initial Access: aws-cli with stolen creds]
    InitialAccess -->|Persistence, add admin policy| PrivEsc[Privilege Escalation: iam PassRole abuse]
    PrivEsc -->|Disable CloudTrail| DefenseEvasion[Defense Evasion]
    DefenseEvasion -->|List buckets and glue tables| Discovery[Discovery]
    Discovery -->|GetObject PHI recursive| Collection[Collection]
    Collection -->|Cross-account copy| Exfiltration[Exfiltration: attacker S3]
    Exfiltration -->|Optional| Impact[Impact: destroy logs, encrypt]
```

## Prerequisites the attacker must hit

- Reach a credential set: leaked PAT in repo, exposed IMDSv1, SSRF from misconfigured app, or a vendor with over-privileged role.
- Egress is allowed from the workload to arbitrary AWS endpoints (typical default).
- Logging is not duplicated to a *separate account* the attacker cannot also disable.

## Why this is plausible at DCS

- Data lake is fed nightly by ETL jobs that often run with broader IAM than the request-path services.
- Older Terraform may leave `*` permissions on S3 prefixes.
- CloudTrail in a single account, no organization-trail to a logging account, is a common single point of failure.

## Most likely controls that exist today (assumed)

- Per-service IAM roles (not user keys) for prod workloads.
- IMDSv2 enforced on EC2.
- CloudTrail enabled in primary region (not org-trail).

## Most likely gaps (to validate during workshop)

- No GuardDuty findings routed to on-call.
- No SCP at organization level preventing `cloudtrail:StopLogging`.
- No anomaly detection on volumetric `s3:GetObject` per role.
- Cross-account S3 copy not denied by SCP.
- Data lake unencrypted with customer-managed KMS (so SSE-S3 alone doesn't gate cross-account).

## Related artifacts

- Detection / response playbook → `Runbook.md`
- MITRE technique walkthrough → `Sequence Summary.md`

## Open questions for the workshop

1. Where is the highest-privilege workload identity, and what does it actually need?
2. Do we have an organization-trail to a separate logging account?
3. Can an SCP block `s3:CopyObject` to non-DCS account IDs at the org level?
4. What is the MTTD for "role X did 10k GetObject in 5 minutes"?
