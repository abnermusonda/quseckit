# QuSecKit

**An open-source, business-first platform for post-quantum cryptography (PQC) remediation.**

QuSecKit runs a PQC migration as a program, not a scan: it inventories the cryptography in your estate, maps every finding to the business service it protects, ranks the work, drives the fix, and closes a finding only when it can prove the new algorithm is in use.

> **Status:** pre-alpha. This repository holds the design ([whitepaper](docs/whitepaper.md)) and will hold the code as the v0.1 milestone lands. Nothing here is ready for production use.

## Why

Discovery tools such as [CBOMkit](https://github.com/pqca/cbomkit) (PQCA / Linux Foundation) already produce a cryptographic bill of materials (CBOM). What is missing in open source is everything after the inventory:

- **Business context:** which RSA-2048 certificate protects the system the business cannot lose?
- **A closed loop:** proof that a fix actually changed what an endpoint negotiates, not just a closed ticket.
- **A shared record:** one place that boards, auditors and regulators can read.

QuSecKit builds that layer on top of CycloneDX CBOM, so it works with the scanners you already run.

## Architecture

```
qsk-connect ──▶ qsk-graph ──▶ qsk-rank
                                  │ ranked queue
                                  ▼
qsk-report ◀── qsk-verify ◀── qsk-fix
     pass         │   ▲
                  └───┘ no evidence of new algorithm: reopen
```

| Module | What it does |
| --- | --- |
| `qsk-connect` | Read-only collectors: Git repos, cloud KMS and certificate managers, PKI/CA, HSM, VPN, IAM, TLS endpoint probes. Ingests CBOMkit output. |
| `qsk-graph` | CycloneDX 1.6 CBOM plus a business-unit → application → service → crypto-asset map (CMDB import or YAML in Git). |
| `qsk-rank` | Scores findings on business impact, data shelf-life (harvest-now-decrypt-later), exposure, compliance deadline and fix effort, with a transparent, editable policy. |
| `qsk-fix` | Turns findings into pull requests, runbooks, Ansible/Terraform changes, or Jira/Linear tickets. |
| `qsk-verify` | Re-probes or re-scans after a fix; closes a finding only on signed evidence. |
| `qsk-report` | Board, auditor and regulator views; migration timeline; NIST FIPS 203/204/205 and CNSA 2.0 mappings. |

## Design principles

1. **Self-hosted first.** Helm chart or Docker Compose; works air-gapped; never holds private keys.
2. **Standards in, standards out.** CycloneDX CBOM is the only inventory format.
3. **Explainable priority.** Every rank shows its inputs; overrides are logged.
4. **Evidence, not status.** "Done" means a verification record exists.
5. **AI optional.** An LLM can draft runbooks and summaries but never sets priority or closes a finding.

## Roadmap

| Version | Target | Scope |
| --- | --- | --- |
| v0.1 | Q1 2027 | CBOMkit ingest, TLS probes, YAML business map, ranked findings UI |
| v0.3 | Q2 2027 | Jira/Linear/GitHub PR integrations, 20 remediation patterns, re-probe verification |
| v0.5 | Q3 2027 | AWS/Azure/GCP KMS and certificate managers, Vault, one HSM vendor, ServiceNow import |
| v1.0 | Q4 2027 | Board and regulator reports, signed evidence export, MCP server |

See the [whitepaper](docs/whitepaper.md) for the full rationale.

## Contributing

Early design feedback is the most useful contribution right now. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[Apache License 2.0](LICENSE)
