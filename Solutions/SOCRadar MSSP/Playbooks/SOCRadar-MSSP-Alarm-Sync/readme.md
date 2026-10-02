# SOCRadar MSSP Alarm Sync

Syncs closed Microsoft Sentinel incidents back to the matching SOCRadar company.

## Features

- Monitors recently closed SOCRadar-labelled incidents whose title starts with `[MSSP-`
- Maps the company back to its `CompanyId` through `CompanyIds` / `CompanyNames`
- Maps Microsoft Sentinel classification to SOCRadar status
- The `Synced` label is added only after SOCRadar confirms the change (`is_success`)
- A refused change writes a `SyncWriteRejected` audit row instead of `IncidentSynced`
- `SyncSeverity` (default off) also pushes the incident severity back on close
- Configurable polling interval
- Optional audit rows: `EnableAuditLogging` plus `AuditDcrEndpoint`, `AuditDcrImmutableId`, `AuditDcrResourceId` of the audit DCR created by the Import deployment

## Classification Mapping

| Microsoft Sentinel Classification | SOCRadar Status |
|------------------------|-----------------|
| FalsePositive | FALSE_POSITIVE |
| BenignPositive | MITIGATED |
| TruePositive | RESOLVED |
| Undetermined | RESOLVED |

## Deployment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadar%20MSSP%2FPlaybooks%2FSOCRadar-MSSP-Alarm-Sync%2Fazuredeploy.json)

You can also install this playbook via **Microsoft Sentinel Content Hub**.

## Prerequisites

- MSSP-scoped SOCRadar API key
- Microsoft Sentinel workspace with SOCRadar MSSP incidents
- Same `CompanyIds` / `CompanyNames` as the Import playbook
- Managed Identity with Microsoft Sentinel Responder (or Contributor) role
