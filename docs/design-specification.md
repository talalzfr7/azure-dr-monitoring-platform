# Design Specification: DR and Monitoring Platform

## Inputs

| Input | Type | Purpose |
|---|---|---|
| `workspaceName` | `string` | Log Analytics workspace name. |
| `retentionInDays` | `int` | Planned default: 30; extend only with an explicit cost/compliance decision. |
| `storageAccountId` | `resourceId` | Source for diagnostic settings. |
| `ingestionThresholdGb` | `int`/`float` | Planned alert threshold: 4.5 GB/day, subject to current billing/API validation. |
| `backupScheduleUtc` | `string` | Planned daily schedule: 02:00 UTC. |
| `dailyRetentionDays` | `int` | Planned value: 30. |
| `weeklyRetentionWeeks` | `int` | Planned value: 12. |
| `monthlyRetentionMonths` | `int` | Planned value: 12. |
| `yearlyRetentionYears` | `int` | Planned value: 5. |
| `sourceRegion` / `targetRegion` | `string` | ASR deployment/test regions. |

## Module responsibilities

### `monitoring.bicep`

Creates the workspace, diagnostic settings for the selected storage account, and the scheduled query alert. It must document the retention and ingestion-threshold decisions inline and avoid embedding destination secrets.

### `backup-policy.bicep`

Creates the Recovery Services vault and the tested backup policy. The implementation must make the retention tiers visible and must not imply that a policy protects a workload until protection and restore have been tested.

### `msp-monitoring-queries.kql`

Contains four independently runnable query blocks with scenario comments. The implementation must identify expected tables/columns and record any schema variation encountered during testing.

## Evidence matrix

| Claim | Required evidence |
|---|---|
| Workspace deployed | Successful deployment output and resource inspection. |
| Logs available | Query against the actual workspace/table with captured result context. |
| Alert works | Alert rule configuration plus a safe test or documented validation path. |
| Backup works | Backup and restore test with workload and policy context. |
| ASR works | Test Failover result, isolated-network verification, and measured RTO. |
| Retention is appropriate | Client requirement/rationale and configured policy output. |
| Access is least privilege | Role assignments, scopes, and workflow justification. |

## Planned repository shape

```text
bicep/
├── backup-policy.bicep
└── monitoring.bicep
kql/
└── msp-monitoring-queries.kql
architecture/
└── dr-monitoring-architecture.png
docs/
├── adr.md
└── design-specification.md
```
