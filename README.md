# Azure DR and Monitoring Platform

> A monitoring, backup, and disaster-recovery design for an MSP client Azure environment.

**Status: design and specification complete; implementation not yet merged.** This public repository records the intended operational architecture, retention decisions, KQL scenarios, and verification criteria. The Bicep, KQL implementation, ASR test results, and observed RTO are intentionally not published yet.

## Client scenario

An MSP client needs 24/7 operational visibility, a tested recovery path, and backup retention that can be explained to an auditor. The platform is designed to make monitoring data queryable, alerts actionable, recovery testable, and retention choices explicit rather than leaving the client with a collection of unconnected portal settings.

## Planned architecture

```text
Azure resources
├── Diagnostic settings ───────┐
├── VM performance/security ───┼──> Log Analytics workspace (30-day default)
└── NSG flow data ─────────────┘              │
                                             ├── KQL investigation queries
                                             ├── Scheduled query alert
                                             └── Dashboard / action group

Protected VMs ──> Recovery Services vault ──> Azure Backup policy
       │
       └──────── asynchronous ASR replication ──> secondary region
                                      │
                              Test Failover + observed RTO
```

## Planned components

| Component | Planned responsibility |
|---|---|
| `bicep/monitoring.bicep` | Log Analytics workspace, diagnostic settings, and ingestion-threshold alert. |
| `bicep/backup-policy.bicep` | Recovery Services vault and daily/weekly/monthly/yearly retention policy. |
| `kql/msp-monitoring-queries.kql` | Four commented queries tied to real MSP investigation scenarios. |
| `docs/adr.md` | Monitoring, DR, backup, security, and operational decisions. |
| `architecture/dr-monitoring-architecture.png` | Diagram after the design and implementation are reviewed. |

## Monitoring design

The baseline uses a Log Analytics workspace with 30-day retention as the default operational window. Diagnostic settings send storage read/write/delete categories to the workspace. A scheduled query rule warns when daily ingestion approaches the planned free-tier boundary; the exact alert implementation must be validated against the active Azure resource API.

The four planned KQL scenarios are:

1. Failed authentication events grouped by source IP and hour.
2. Storage account access outside the defined business-hour window.
3. Daily VM CPU averages and peaks for capacity planning.
4. NSG flow denies grouped by source, destination port, and day.

The comments in the eventual query file must explain the client scenario and how an operator should interpret the output. A query is not operational evidence until it has been run against the relevant table/schema.

## DR and backup design

- Azure Site Recovery is the planned Azure-to-Azure replication and failover mechanism.
- A Test Failover is required before any claim about recoverability; it must use an isolated test network and leave the source workload running.
- The observed RTO is measured from failover initiation to verified VM accessibility. No RTO value is pre-filled in this repository.
- The planned backup policy retains daily recovery points for 30 days, weekly points for 12 weeks, monthly points for 12 months, and yearly points for five years. These are design targets, not proof of a configured policy.
- Retention should be adjusted when the client's regulatory and operational requirements differ.

## Security and least privilege

The planned access model distinguishes operational roles: Monitoring Reader for read-only dashboard access, Log Analytics Contributor for automation that must configure workspace resources, and Backup Operator for backup administration. The implementation must scope these roles as narrowly as the actual workflow permits; broad Contributor access is not the default.

## Verification criteria

1. Bicep compiles for monitoring and backup modules.
2. Diagnostic data arrives in the workspace and each query is tested against the actual tables available in the deployment.
3. The ingestion alert is configured and its action path is documented.
4. Backup and restore behaviour is tested with a disposable workload.
5. ASR Test Failover completes in an isolated network and the observed RTO is recorded.
6. Replication, vaults, and other credit-consuming test resources are cleaned up and verified.
7. The README separates planned targets, observed measurements, and unresolved implementation questions.

## Evidence boundary

This repository currently proves that the operational design has been specified. It does **not** yet prove that a workspace receives data, that the KQL queries run, that backup restores, or that ASR has produced an observed RTO.

## AZ-104 and MSP relevance

The project maps the smallest AZ-104 domain to the part of MSP work clients feel most directly: detection, investigation, recoverability, and evidence that a recovery promise was tested rather than merely documented.

- Sprint scope: SD-54, SD-58, and SD-59.
- Exam domain: Monitor and maintain Azure resources.
- Design references: [Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview), [Log Analytics workspaces](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview), [Azure Backup](https://learn.microsoft.com/en-us/azure/backup/backup-overview), and [ASR Azure-to-Azure architecture](https://learn.microsoft.com/en-us/azure/site-recovery/azure-to-azure-architecture).
- Study-plan source: [AZ-104 sprint repository](https://github.com/talalzfr7/az104-sprint).

See [`docs/adr.md`](docs/adr.md) for the decisions and [`docs/design-specification.md`](docs/design-specification.md) for the implementation and evidence contract.
