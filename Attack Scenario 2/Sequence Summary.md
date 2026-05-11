# Scenario 2 — MITRE ATT&CK sequence

## Narrative

An attacker discovers a developer's personal repo contains a `.env.backup` with an AWS access key (T1552.001). The key belongs to a CI service principal in DCS's prod account with `s3:*` on the analytics bucket. The attacker assumes the role, expands persistence, disables logging, then exfiltrates terabytes of analytics data to their own S3 bucket.

## Sequence

```mermaid
sequenceDiagram
    participant Att as Attacker
    participant Recon as OSINT GitHub
    participant AWS as DCS AWS account
    participant CT as CloudTrail
    participant S3 as Analytics S3
    participant ExfilS3 as Attacker S3

    Att->>Recon: Search public repos for env files and AWS keys
    Recon-->>Att: Leaked AWS access key T1552.001
    Att->>AWS: sts GetCallerIdentity T1078.004
    AWS-->>Att: Identity confirmed role ci-deploy
    Att->>AWS: iam CreateAccessKey for self T1098.001
    Att->>AWS: iam AttachRolePolicy AdministratorAccess T1098.003
    Att->>AWS: cloudtrail StopLogging T1562.008
    AWS-->>CT: Trail stopped, gap begins
    Att->>AWS: s3 ListBuckets T1580
    AWS-->>Att: dcs-phi-analytics, dcs-phi-attachments
    Att->>S3: s3 GetObject recursive T1530
    S3-->>Att: PHI rows streamed
    Att->>ExfilS3: s3 PutObject cross-account T1567.002
    Att->>AWS: cloudtrail DeleteTrail T1485 optional
```

## Detection opportunities (in order)

1. **GitHub secret-scanning** — should fire on the original push of `.env.backup`. If push protection is on, the attack is stopped here.
2. **GuardDuty `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`** — fires when an access key tied to an EC2 role is used from outside that EC2 (common pattern for stolen IMDS).
3. **CloudTrail `StopLogging` event** — should page security immediately; this is the high-confidence indicator of an in-progress incident.
4. **S3 server-access logs / CloudTrail data events** — bulk `GetObject` volume from one principal, in a short window, on a PHI bucket, is anomalous and should alert.
5. **VPC flow / NetFlow** — high outbound bytes from analytics workload toward non-DCS S3 endpoints.

If none of (1)–(5) fire, the attack succeeds undetected. That is the gap to close.

## MITRE technique table

| Step | Technique | ID |
|------|-----------|-----|
| Find leaked key | Credentials in Files | T1552.001 |
| Use cloud creds | Valid Accounts: Cloud | T1078.004 |
| Self-issue new key | Account Manipulation | T1098.001 |
| Grant admin policy | Cloud Roles | T1098.003 |
| Disable trail | Disable Cloud Logs | T1562.008 |
| List buckets | Cloud Infra Discovery | T1580 |
| Read PHI objects | Data from Cloud Storage | T1530 |
| Copy to attacker S3 | Exfil to Cloud Storage | T1567.002 |
| Delete trail | Data Destruction | T1485 |

## Hypotheses to test in the workshop

- (H1) GitHub secret-scanning is enabled with push protection on every DCS-owned org. → If false, R8 should be raised in severity.
- (H2) GuardDuty is enabled in every region and findings are routed to on-call. → If false, this scenario gets a near-100% success probability.
- (H3) `cloudtrail:StopLogging` is denied by an SCP at the org level. → If false, this is the single most cost-effective control to add.
