# SOCRadar Entra ID Integration for Microsoft Sentinel

The **SOCRadar Entra ID Integration** automatically responds to leaked employee credentials detected by SOCRadar's Dark Web Monitoring platform. It pulls Botnet Data, PII Exposure, and VIP Protection alerts from the SOCRadar XTI Platform, looks up matching identities in Microsoft Entra ID, and takes configurable remediation actions to contain the compromise.

## What it does

```
SOCRadar Platform (3 sources)
       │
       ▼
 Azure Function App (Python 3.11)
       │
   ┌───┴────────┐
   │            │
Microsoft   Microsoft Sentinel
Graph API   (Log Analytics)
─────────   ──────────────────
Look up     SOCRadar_Botnet_CL
Revoke      SOCRadar_PII_CL
Disable     SOCRadar_VIP_CL
MFA wipe    SOCRadar_EntraID_Audit_CL
Group add   4 Workbooks
Password    Sentinel incidents
reset
```

## Data sources

| Source | Description |
|--------|-------------|
| **Botnet Data v2** | Employee credentials harvested from botnet logs |
| **PII Exposure v2** | Employee credentials from third-party data breaches |
| **VIP Protection v2** | VIP users mentioned across dark web sources |

## Remediation actions (toggle-driven)

All actions are disabled by default except user lookup, session revocation, and group membership. Customers enable additional actions based on their security policy and Microsoft Graph permissions granted:

- Revoke active sign-in sessions
- Add to quarantine security group / Remove from group
- Disable / Re-enable account
- Force password change on next sign-in
- Force MFA re-registration (deletes all non-password authentication methods)
- Mark user as compromised in Microsoft Entra ID Protection (requires P1/P2)
- Create Microsoft Sentinel incident
- Resolve SOCRadar alarm when remediated

## Authentication

**Secretless** via Workload Identity Federation (UAMI + Federated Identity Credential → App Registration). No client secret to rotate, no shared key to store.

## Data ingestion

Uses the modern **Data Collection Rule (DCR) — Logs Ingestion API**, Microsoft's replacement for the deprecated HTTP Data Collector API (incident support deadline: Sep 14, 2026).

## Workbooks installed

1. **SOCRadar Entra ID — Botnet Data** — Botnet leak detail + Entra ID match status
2. **SOCRadar Entra ID — PII Exposure** — Breach-sourced credentials + remediation outcome
3. **SOCRadar Entra ID — VIP Protection** — VIP keyword mentions on dark web
4. **SOCRadar Entra ID — Combined Dashboard** — Aggregate view across all sources + audit summary

## Playbook (Azure Function App) installed

A single Python 3.11 Function App with timer trigger (default every 6 hours) that:

1. Acquires Microsoft Graph token via UAMI + Federated Identity Credential
2. Polls Botnet / PII / VIP endpoints on the SOCRadar Platform
3. Filters records to employees (server-side flag) and sanitizes passwords
4. For each leaked credential, looks up the user in Entra ID via Microsoft Graph
5. If found, executes the configured action chain (revoke, disable, MFA wipe, etc.)
6. Writes per-source records and audit summary to Log Analytics via DCR Logs Ingestion API
7. Optionally creates Microsoft Sentinel incidents for compromised users
8. Optionally resolves the originating SOCRadar alarm via the SOCRadar API

## Prerequisites

- Azure subscription with Microsoft Sentinel-enabled Log Analytics workspace
- SOCRadar Platform API key + Company ID
- App Registration with Microsoft Graph application permissions (granted by tenant admin)
- For VIP Protection source: confirm with SOCRadar that the VIP endpoint is enabled for your company

## Configurable parameters

The ARM template exposes ~25 parameters covering source toggles, action toggles, polling schedule, lookback window, and Sentinel integration. Defaults are conservative (high-impact actions disabled). Full parameter reference: see the standalone repository at https://github.com/orcunsami/SOCRadar-Azure-Entra-ID.

## Microsoft Graph permissions required

| Permission | Required for |
|------------|--------------|
| `User.Read.All` | User lookup |
| `User.RevokeSessions.All` | Revoke sessions |
| `GroupMember.ReadWrite.All` | Group add/remove |
| `User-PasswordProfile.ReadWrite.All` | Force password change |
| `User.EnableDisableAccount.All` | Disable / Enable account |
| `IdentityRiskyUser.ReadWrite.All` | Confirm compromised (requires Entra ID P1/P2) |
| `UserAuthenticationMethod.ReadWrite.All` | Force MFA re-registration (also needs Privileged Authentication Administrator role) |

Customers grant only the permissions needed for the actions they enable (least-privilege).

## Support

- Email: integration@socradar.io
- SOCRadar Platform docs: https://docs.socradar.io
- Standalone GitHub repo (with detailed customer guide): https://github.com/orcunsami/SOCRadar-Azure-Entra-ID
