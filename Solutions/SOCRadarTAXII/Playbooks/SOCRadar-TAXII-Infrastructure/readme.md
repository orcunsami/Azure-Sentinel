# SOCRadar-TAXII-Infrastructure

Creates custom log table and data collection infrastructure for SOCRadar TAXII 2.1 audit logging.

## What It Creates

- **Data Collection Endpoint** (SOCRadar-TAXII-DCE)
- **Custom Log Table** (SOCRadar_TAXII_Audit_CL) with columns:
  - TimeGenerated, ApiRoot, CollectionId
  - IndicatorsCreated, IndicatorsSkipped, IndicatorsFailed, IndicatorsRevoked
  - PagesFetched, DurationMs, Status (Success, PartialSuccess, Failed), ErrorMessage
- **Data Collection Rule** (SOCRadar-TAXII-Audit-DCR)

## Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| WorkspaceName | Log Analytics Workspace name | - |
| WorkspaceLocation | Workspace location | Resource group location |
| RetentionDays | Log retention (7-730 days) | 30 |

## Prerequisites

- Existing Log Analytics Workspace with Microsoft Sentinel
- The SOCRadar-TAXII-FunctionApp playbook creates the same resources when `EnableAuditLogging` is true; use this playbook only when you deploy the audit infrastructure separately
