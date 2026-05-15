<img src="logo/socradar.svg" alt="SOCRadar" width="75px" height="75px">

# SOCRadar MSSP Intelligence for Microsoft Sentinel — **Content Hub Solution**

> This is the **Content Hub Solution** package source (Microsoft Sentinel Marketplace path). For the **Standalone Deploy-to-Azure** source, see [`../Seperate-Repo/README.md`](../Seperate-Repo/README.md).

## Overview

The SOCRadar MSSP solution for Microsoft Sentinel monitors **multiple SOCRadar Company IDs from a single Microsoft Sentinel deployment** — for MSSPs, holding companies, and any organisation that subscribes to SOCRadar for several legal entities under one Microsoft Sentinel workspace. Alarms come in as Microsoft Sentinel incidents tagged by company; closed Microsoft Sentinel incidents sync back to the matching SOCRadar account.

For single-customer deployments, use the standard `SOCRadar` solution instead.

## Key Features

**Multi-company alarm import**
- One Logic App polls SOCRadar per company per cycle. Configurable via `CompanyIds` + `CompanyNames` CSV parameters.
- Incident title prefixed with `[<CompanyName>] `; tagged with `Company:<CompanyName>`.
- Per-company deduplication against existing Microsoft Sentinel incidents (composite `(company_id, alarm_id)` key).
- Per-company error isolation — one company's API failure logs to audit but doesn't block other companies.

**Bidirectional sync**
- Closed Microsoft Sentinel incidents parsed back to their `(company_id, alarm_id)` pair via the deploy-time `CompanyIds`/`CompanyNames` mapping.
- Classification mapping: `TruePositive` → Resolved, `FalsePositive` → False Positive, `BenignPositive` → Mitigated.

**Analytics**
- SOCRadar MSSP Dashboard workbook with Company Breakdown tile + CompanyName filter dropdown.
- 5 hunting queries (Company Breakdown, Critical Alarms, Alarm Overview, Alarm Trends, Audit Analysis).
- 3 analytic rules grouped by company (Volume Spike, Critical Alarm Detection, Unsynced Closed Incident).

**Operational visibility**
- Custom LAW tables `SOCRadar_MSSP_Alarms_CL` + `SOCRadar_MSSP_AuditLog_CL` with `company_name` column for KQL pivoting.
- Per-cycle per-company audit row (`Severity_s = INFO` / `ERROR`).
- DCR-based Logs Ingestion API (HTTP Data Collector deprecation handled).

## Prerequisites

- Microsoft Sentinel workspace.
- **MSSP-scoped SOCRadar API key** — one key that grants access to every Company ID monitored. Configured by the SOCRadar account team.
- Comma-separated list of SOCRadar `CompanyIds` and matching `CompanyNames`.

## Installation

Install from the Microsoft Sentinel **Content Hub**:

1. Microsoft Sentinel → **Content Hub**.
2. Search for **SOCRadar MSSP**.
3. Select → **Install**.
4. From the Solution's **Manage** view, deploy:
   - **SOCRadar-MSSP-Alarm-Import** — supply `CompanyIds`, `CompanyNames`, `SOCRadarAPIKey`, `WorkspaceName`. Set `EnableAlarmsTable=true` and `EnableAuditLogging=true` if you want the per-company KQL pivot (recommended).
   - **SOCRadar-MSSP-Alarm-Sync** — same `CompanyIds`/`CompanyNames` so it can reverse-map incident tags to Company IDs.

Both playbooks authenticate to Microsoft Sentinel via Azure Managed Identity. Logic Apps begin polling 3 minutes after deployment.

## End-to-End Test Status (Content Hub path)

Live E2E test on Azure (2026-05-15 NZST):

- **Content Hub Import playbook greenfield deploy** (fresh workspace, CompanyIds=`132,200,300`, `EnableAlarmsTable=true`, `EnableAuditLogging=true`) → `Succeeded` in 34s
- **Logic App run**: For_Each_Company iterated 3 companies, Ingest_To_Custom_Table **2/2 Succeeded** for Gamma (2 alarms)
- **LAW `SOCRadar_MSSP_Alarms_CL`** + **LAW `SOCRadar_MSSP_AuditLog_CL`**: rows ingested with `company_name=Gamma`
- **Microsoft Sentinel Incidents**: 2 incidents created in the CH workspace with `[MSSP-Gamma]` title prefix
- **Idempotency**: re-trigger → 0 new incidents (composite dedup confirmed)

Same source, two install paths. See [`../to-Radargoger/TESLIM-MSSP-Incidents.md`](../to-Radargoger/TESLIM-MSSP-Incidents.md) for the full evidence table.

## Playbooks

| Playbook | Description |
|----------|-------------|
| [SOCRadar-MSSP-Alarm-Import](Playbooks/SOCRadar-MSSP-Alarm-Import) | Multi-company import. Provisions DCE + 2 custom tables + 2 DCRs + role assignments (Option A). |
| [SOCRadar-MSSP-Alarm-Sync](Playbooks/SOCRadar-MSSP-Alarm-Sync) | Multi-company sync back to SOCRadar based on the `Company:<name>` tag. |

## About SOCRadar

SOCRadar is an Extended Threat Intelligence (XTI) platform providing actionable threat intelligence, digital risk protection, and external attack surface management.

Learn more at [socradar.io](https://socradar.io)

## Support

- **Documentation:** [docs.socradar.io](https://docs.socradar.io)
- **Support:** integration@socradar.io
