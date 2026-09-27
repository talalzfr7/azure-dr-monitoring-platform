# Architecture Decision Record: DR and Monitoring Platform

- **Status:** Design and specification complete; implementation not yet merged
- **Date:** 2026-09-27
- **Scope:** MSP monitoring, backup, and Azure-to-Azure disaster recovery
- **Implementation status:** Not yet merged or deployed

## Context

A client environment can be well-designed and still be operationally unsafe if nobody can see failures, restore data, or demonstrate that the recovery path works. The platform therefore treats observability, backup, and disaster recovery as one operational system with separate evidence requirements.

## Decision 1: Log Analytics as the monitoring hub

Log Analytics is the central store for the log data selected by the design. A 30-day default retention balances routine investigation with cost control; a regulated client may require a longer period, which must be an explicit parameter and cost decision rather than a silent default.

Diagnostic settings send StorageRead, StorageWrite, and StorageDelete categories from a parameterised storage account. Additional VM, security, or NSG data sources may be added once their table/schema and cost are validated.

## Decision 2: Alert through scheduled queries and action groups

The first alert is an ingestion guard: it warns when daily workspace ingestion approaches the planned free-tier boundary. The alert is useful operationally because an observability system that silently exceeds its budget is not a reliable control. The action group and notification path must be configured as part of implementation; the design does not claim that a notification has fired yet.

## Decision 3: KQL queries are scenario-led

The query set is organised around four MSP workflows: failed-authentication investigation, off-hours storage access, CPU capacity planning, and NSG deny review. Each query must carry a comment block explaining what question it answers and what an operator should do with the output.

The design forbids treating a plausible query as evidence. Table names, columns, and event availability must be checked against the actual workspace before a query is described as working.

## Decision 4: Azure Site Recovery for tested DR

Azure Site Recovery is the planned Azure-to-Azure replication mechanism. The required test is Test Failover into an isolated network; the source workload remains running. The RTO is measured from the recorded initiation point to verified service accessibility. The repository must record the observed value and test conditions; it must not substitute a vendor typical range or a target for an observation.

A weekly Test Failover cadence is a recommendation for the client operating model, subject to cost, workload, and change-control review. RPO is tied to replication lag and must be stated with the workload and measurement context.

## Decision 5: Tiered backup retention

The planned policy is:

- Daily recovery points for 30 days: operational recovery.
- Weekly recovery points for 12 weeks: version rollback and medium-term recovery.
- Monthly recovery points for 12 months: quarterly/audit-oriented recovery.
- Yearly recovery points for 5 years: regulatory retention target where the client requires it.

These values are design targets. The implementation must verify that the chosen vault, policy API, and workload support them and must adjust them to the client's retention obligations.

## Decision 6: Least-privilege operational roles

Monitoring Reader is the planned read-only role for dashboards and investigation. Log Analytics Contributor is reserved for automation that must configure workspace resources. Backup Operator is reserved for backup administration. These are intentionally narrower than Contributor; the actual assignment scope must match the workflow and be reviewed after implementation.

## Planned deliverable

A completed project should provide deployable Bicep, four scenario-led KQL queries, an architecture diagram, backup/restore and ASR test evidence, observed RTO, cleanup records, and an ADR that lets a client or reviewer distinguish target, configuration, and observation.

## Open decisions

- Final diagnostic categories and workspace table availability for each query.
- Exact scheduled-query alert API shape and action-group destination.
- Target workload and isolated test network for the ASR test.
- RPO target and acceptable RTO agreed with the client before the test.
- Whether five-year retention is required, affordable, and legally appropriate for the chosen client scenario.
- Current costs and teardown order for Recovery Services vault, ASR, and protected resources.
- Safe validation of the ingestion alert; the 4.5 GB threshold is specified but has not fired in an observed deployment.
