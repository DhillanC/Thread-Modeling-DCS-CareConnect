# Index — DCS Care Connect Health 360 Threat Model

Single entry point to every artifact in this repository. Use this when onboarding a new reviewer, auditor, or engineer.

## Top-level

- [README.md](README.md) — workshop summary, scope, methodology, controls.
- [RISK SUMMARY.md](RISK%20SUMMARY.md) — risk register (R1–R8) with STRIDE, MITRE, HIPAA mapping, owners, timelines.

## Application Summary (architecture & posture)

- [High Level Design.md](Application%20Summary/High%20Level%20Design.md) — component-level architecture diagram.
- [Data Flow Diagram.md](Application%20Summary/Data%20Flow%20Diagram.md) — DFD with trust boundaries, data classification per flow.
- [Inherent Risk Assessment.md](Application%20Summary/Inherent%20Risk%20Assessment.md) — pre-control risk picture (data sensitivity, volume).
- [STRIDE Analysis.md](Application%20Summary/STRIDE%20Analysis.md) — STRIDE per component with MITRE techniques and controls.
- [Asset Classification.md](Application%20Summary/Asset%20Classification.md) — data tiers, patient criticality tiers, BIA.
- [Compliance Mapping.md](Application%20Summary/Compliance%20Mapping.md) — HIPAA Security Rule mapping, breach notification SLA.

## Attack scenarios

Each scenario has three artifacts: a narrative README, an ATT&CK sequence summary, and a blue-team runbook.

- **Scenario 1 — AI-generated phishing → admin credential compromise**
  - [README.md](Attack%20Scenario%201/README.md)
  - [Sequence Summary.md](Attack%20Scenario%201/Sequence%20Summary.md)
  - [Attacker Flow.md](Attack%20Scenario%201/Attacker%20Flow.md)
  - [Runbook.md](Attack%20Scenario%201/Runbook.md)

- **Scenario 2 — Cloud credential abuse → data-lake exfiltration**
  - [README.md](Attack%20Scenario%202/README.md)
  - [Sequence Summary.md](Attack%20Scenario%202/Sequence%20Summary.md)
  - [Runbook.md](Attack%20Scenario%202/Runbook.md)

- **Scenario 3 — SQL injection against PHI database**
  - [README.md](Attack%20Scenario%203/README.md)
  - [Sequence Summary.md](Attack%20Scenario%203/Sequence%20Summary.md)
  - [Runbook.md](Attack%20Scenario%203/Runbook.md)

- **Scenario 4 — Insider threat: proprietary IP / data exfiltration**
  - [README.md](Attack%20Scenario%204/README.md)
  - [Sequence Summary.md](Attack%20Scenario%204/Sequence%20Summary.md)
  - [Runbook.md](Attack%20Scenario%204/Runbook.md)

## Archive (reference only — not authoritative)

- [Archive/STRIDE.md](Archive/STRIDE.md) — generic STRIDE template (superseded by `Application Summary/STRIDE Analysis.md`).
- [Archive/Controls Required.md](Archive/Controls%20Required.md) — early controls list (superseded by `RISK SUMMARY.md` mitigation columns).

## How to use this repo

1. **Auditor / new reviewer**: start at [README.md](README.md), then [RISK SUMMARY.md](RISK%20SUMMARY.md), then [Application Summary/Compliance Mapping.md](Application%20Summary/Compliance%20Mapping.md).
2. **Engineer on a ticket**: jump to the relevant `Attack Scenario *` and look up the controls it expects in `STRIDE Analysis.md`.
3. **SOC / on-call**: every alert should map to one of the `Attack Scenario */Runbook.md` files.
4. **Quarterly review**: walk `RISK SUMMARY.md` end-to-end, update statuses, refresh the open-questions sections in each scenario.

## Quality checklist for changes

Before merging a change to this repo:

- [ ] Markdown lints clean (`markdownlint`).
- [ ] All Mermaid diagrams render (`mermaid-cli --input file.md`).
- [ ] No broken internal links.
- [ ] Image paths use forward slashes (`Archive/dcs_logo.png`, not `Archive\dcs_logo.png`).
- [ ] If a risk is added or modified, the JIRA ticket reference is included.
- [ ] If a control is described as "in place", evidence vault location is named.
