# QuSecKit: An Open-Source, Business-First PQC Remediation Platform

*Whitepaper draft, 2026-11-23*

## Executive summary

QuSecKit is a proposed Apache-2.0 platform that runs post-quantum cryptography (PQC) migration as a business-prioritized program: inventory, risk ranking, fixes and verification in one open stack. It targets the same job as the commercial PQC remediation platforms now raising venture rounds.

The open-source gap is clear. Discovery is already commoditized by the Linux Foundation's CBOMkit, but nothing open connects a CBOM to business risk, drives the fix and proves it closed. QuSecKit builds that layer on CycloneDX CBOM so it composes with existing scanners rather than competing with them.

The bet: banks, insurers and governments facing 2030–2035 deadlines will prefer a self-hosted, auditable control plane for their most sensitive asset (key material) over a seed-stage SaaS vendor.

## The problem

Most PQC programs stall after the inventory: teams get thousands of findings and no defensible order to fix them in. A US executive order signed 2026-06-22 requires federal high-value assets to use post-quantum key establishment by end of 2030 and post-quantum signatures by end of 2031, and regulated industries are following.

Three gaps keep migrations stuck:

- **No business context.** Scanners report "RSA-2048 in repo X"; they cannot say it protects the payments API the business cannot lose.
- **No closed loop.** Findings become tickets that get closed without anyone proving the endpoint now negotiates ML-KEM.
- **No trusted shared record.** Boards, auditors and regulators each want a different view of progress, so teams rebuild spreadsheets every quarter.

Commercial platforms address these gaps, but they ask a bank to grant a young vendor read access to its PKI, HSM and KMS estate.

## Competitive landscape

The market splits into funded commercial platforms that span the whole program and open-source tools that stop at discovery.

| Player | Model | Scope | Funding / status | What QuSecKit takes from it |
| --- | --- | --- | --- | --- |
| [QIZ Security](https://www.securityweek.com/qiz-security-raises-17-million-for-cryptographic-governance-platform/amp/) | Commercial | Cryptographic posture and PQC management | $17M seed, July 2026 | Continuous posture, not one-time assessment |
| [QuSecure](https://www.securityweek.com/qusecure-banks-28m-series-a-for-post-quantum-cryptography-tech/amp/) | Commercial, AWS Marketplace | Network overlay that swaps algorithms without app rewrites | $28M Series A | A remediation target to integrate with, not compete against |
| [CBOMkit (PQCA / Linux Foundation)](https://github.com/pqca/cbomkit) | Open source, Apache-2.0 | Source and container scanning to CycloneDX CBOM | Donated by IBM Research | The discovery layer QuSecKit consumes |

QuSecKit's opening: CBOMkit ends at "here is your inventory"; the commercial platforms begin there but are closed. No open project owns the risk graph, prioritization and verified-fix loop.

## Architecture

```
qsk-connect ──▶ qsk-graph ──▶ qsk-rank
                                  │ ranked queue
                                  ▼
qsk-report ◀── qsk-verify ◀── qsk-fix
     pass         │   ▲
                  └───┘ no evidence of new algorithm: reopen
```

Findings flow from read-only collection to a ranked queue; a fix that verification cannot confirm goes back to `qsk-fix` instead of closing.

## Core modules

Six modules, each usable alone, all sharing one CycloneDX 1.6 CBOM plus a business-context graph.

| Module | What it does | Built on |
| --- | --- | --- |
| `qsk-connect` | Read-only collectors for GitHub/GitLab, AWS/Azure/GCP KMS and certificate managers, PKI/CA, HSM, VPN, IAM, TLS endpoint probes | Ingests CBOMkit, testssl/sslyze and cloud APIs; plugin SDK for new sources |
| `qsk-graph` | Maps business unit → application → service → crypto asset; imports from CMDB (ServiceNow) or a YAML file in Git | Graph store (Postgres + recursive queries to start) |
| `qsk-rank` | Scores each finding on business impact, data shelf-life (harvest-now-decrypt-later), exposure, compliance deadline and fix effort | Transparent, editable scoring policy in code; no black-box AI score |
| `qsk-fix` | Turns findings into runbooks, pull requests, Ansible/Terraform changes or Jira/Linear tickets with change-control metadata | Template library per pattern (TLS config, library upgrade, cert reissue, hybrid ML-KEM) |
| `qsk-verify` | Re-probes or re-scans after a fix; a finding closes only when evidence shows the new algorithm in use | Same collectors as `qsk-connect`; signed evidence records |
| `qsk-report` | Board, auditor and regulator views; migration timeline by workstream; NIST FIPS 203/204/205 and CNSA 2.0 mappings | Static exports plus an optional LLM assistant over an MCP server |

Design principles:

1. **Self-hosted first.** Runs as a Helm chart or one Docker Compose file, works air-gapped, never holds private keys.
2. **Standards in, standards out.** CycloneDX CBOM is the only inventory format, so any scanner can feed it and any GRC tool can read it.
3. **Explainable priority.** Every rank shows its inputs; risk owners can override and the override is logged.
4. **Evidence, not status.** "Done" means a verification record exists, not that a ticket moved.
5. **AI optional.** The LLM layer drafts runbooks and summaries but never decides priority or closes a finding.

## Open-source strategy and sustainability

QuSecKit wins on trust and price, the two places a closed seed-stage vendor is weakest.

- **License:** Apache-2.0 for the core, matching CBOMkit and keeping the door open to a Linux Foundation / PQCA home once adoption justifies it.
- **Positioning:** the program layer on top of CBOMkit, not a rival scanner. Contributing upstream earns credibility with the PQCA community.
- **Free forever:** all six modules, single-tenant, unlimited assets.
- **Paid (open-core or services):** multi-tenant SSO/RBAC and audit packs for regulated buyers, hosted QuSecKit Cloud, and a fixed-price migration assessment that undercuts commercial pilots.
- **Distribution:** GitHub Action and Helm chart first, AWS/Azure Marketplace listings second, so security teams can try it before procurement is involved.
- **Community:** a public remediation-pattern library (one YAML per fix pattern) is the contribution magnet, the same way detection rules drive Sigma and Semgrep.

## Roadmap

1. **v0.1, Q1 2027 — Inventory + graph.** Ingest CBOMkit output and TLS probes; YAML business map; ranked findings in a web UI. Gate: three design-partner teams load real data.
2. **v0.3, Q2 2027 — Fix + verify loop.** Jira/Linear/GitHub PR integrations, 20 remediation patterns, re-probe verification. Gate: first verified ML-KEM hybrid rollout at a partner.
3. **v0.5, Q3 2027 — Cloud and PKI depth.** AWS/Azure/GCP KMS and certificate managers, HashiCorp Vault, one HSM vendor, ServiceNow CMDB import.
4. **v1.0, Q4 2027 — Governance.** Board and regulator reports, CNSA 2.0 and NIST mappings, signed evidence export, MCP server for AI assistants.
5. **2028 — Ecosystem.** Propose QuSecKit to PQCA; certify pattern packs with integrators.

## Risks

The biggest risk is scope: funded commercial vendors have money and teams; an open project must ship the fix-and-verify loop narrowly before adding breadth.

- **Connector depth.** HSM and mainframe access is where commercial vendors earn their price; lean on partners and the plugin SDK.
- **Upstream overlap.** PQCA may extend CBOMkit toward prioritization; engage early so QuSecKit is the complement, not the duplicate.
- **Buyer trust.** Regulated buyers want a vendor behind open source; the paid tier and a foundation home answer that.
- **Name.** QuSecKit sits close to QuSecure, an established PQC vendor; run a trademark search before a public launch.

## Sources

- [SecurityWeek: QIZ Security raises $17M](https://www.securityweek.com/qiz-security-raises-17-million-for-cryptographic-governance-platform/amp/)
- [SiliconANGLE: QIZ seed and the June 2026 executive order](https://siliconangle.com/?p=787753)
- [SecurityWeek: QuSecure $28M Series A](https://www.securityweek.com/qusecure-banks-28m-series-a-for-post-quantum-cryptography-tech/amp/)
- [IBM Research: CBOMkit donated to the Linux Foundation](https://researcher.ibm.com/blog/cryptographic-cbom-linux-foundation)
- [PQCA CBOMkit on GitHub](https://github.com/pqca/cbomkit)
