| **Version** | **Date Modified (DD-MM-YYYY)** | **Change History** |
|-------------|--------------------------------|--------------------|
| 3.0.0       | 01-10-2026                     | Function App playbook matches the standalone integration: first-run window (`InitialLookbackDays`), package staged in the storage account by the deployment script, Microsoft Sentinel Contributor role scoped to the workspace, hardened storage and app settings. Audit table gains `CollectionsFailed` and `IndicatorsFailed` and the `PartialSuccess` status; the workbook indicator counts and the Overview and Collection-Stats hunting queries count distinct indicators (the workbook timeline and Top-Indicators still count rows). |
| 2.1.0       | 09-03-2026                     | Added Application Insights monitoring, automatic first run via deployment script, step-by-step logging. |
| 2.0.0       | 09-03-2026                     | Migrated from Logic App to Azure Function App with batch upload and User-Assigned Managed Identity. |
| 1.0.0       | 27-02-2026                     | Initial release.   |
