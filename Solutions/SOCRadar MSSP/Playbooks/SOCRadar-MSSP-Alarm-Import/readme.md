# SOCRadar MSSP Alarm Import

Imports SOCRadar alarms for several companies into Microsoft Sentinel as incidents. One Logic App polls SOCRadar per company per cycle.

## Features

- `CompanyIds` and `CompanyNames` (CSV, same length, matched by position)
- Incident title `[MSSP-<CompanyName>] #<alarm_id> - <alarm_title>`, label `MSSP-<CompanyName>`
- Paginated alarm fetching (100 per page), processed page by page
- Duplicate detection against existing Microsoft Sentinel incidents, per company
- A failed or incomplete page terminates the run `Failed` (`ImportIncomplete`)
- One company's API failure is logged to the audit table and does not block the others
- Severity and status mapping; optional import of closed alarms
- Optional audit logging and alarm table (`SOCRadar_MSSP_AuditLog_CL`, `SOCRadar_MSSP_Alarms_CL`)

## Prerequisites

- MSSP-scoped SOCRadar API key (covers every Company ID listed)
- Microsoft Sentinel workspace
- Managed Identity with Microsoft Sentinel Responder (or Contributor) role

## Deployment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadar%20MSSP%2FPlaybooks%2FSOCRadar-MSSP-Alarm-Import%2Fazuredeploy.json)

You can also install this playbook via **Microsoft Sentinel Content Hub**.
