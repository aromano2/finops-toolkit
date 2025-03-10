---
layout: default
title: Common errors
nav_order: 888
description: 'Details and solutions for common issues you may experience.'
permalink: /help/errors
---

<span class="fs-9 d-block mb-4">Troubleshooting common errors</span>
Sorry to hear you're having a problem. We're here to help!
{: .fs-6 .fw-300 }

<details markdown="1">
   <summary class="fs-2 text-uppercase">On this page</summary>

- [BadHubVersion](#badhubversion)
- [InvalidExportContainer](#invalidexportcontainer)
- [InvalidExportVersion](#invalidexportversion)
- [InvalidHubVersion](#invalidhubversion)
- [InvalidScopeId](#invalidscopeid)
- [ExportNotFound](#exportnotfound)
- [NoDataIngested](#nodataingested)
- [ManifestReadFailed](#manifestreadfailed)
- [ResourceAccessForbiddenException](#resourceaccessforbiddenexception)
- [RoleAssignmentUpdateNotPermitted](#roleassignmentupdatenotpermitted)
- [SchemaLoadFailed](#schemaloadfailed)
- [SchemaNotFound](#schemanotfound)
- [UnknownExportFile](#unknownexportfile)
- [UnknownHubVersion](#unknownhubversion)
- [UnsupportedExportType](#unsupportedexporttype)
- [The \<name\> resource provider is not registered in subscription \<guid\>](#the-name-resource-provider-is-not-registered-in-subscription-guid)
- [x\_PricingSubcategory shows the commitment discount ID](#x_pricingsubcategory-shows-the-commitment-discount-id)
- [Power BI: Reports are empty (no data)](#power-bi-reports-are-empty-no-data)
- [Power BI: The remote name could not be resolved: '\<storage-account\>.dfs.core.windows.net'](#power-bi-the-remote-name-could-not-be-resolved-storage-accountdfscorewindowsnet)
- [Power BI: We cannot convert the value null to type Logical](#power-bi-we-cannot-convert-the-value-null-to-type-logical)
- [FinOps hubs: We cannot convert the value null to type Table](#finops-hubs-we-cannot-convert-the-value-null-to-type-table)
- [Next steps](#next-steps)

</details>

---

This article describes common FinOps toolkit errors and provides information about solutions. If you get an error when using FinOps toolkit solutions that you don't understand or can't resolve, find the error code below with mitigation steps to resolve the problem.

If the information provided doesn't resolve the issue, try the [Troubleshooting guide](./troubleshooting.md).

<br>

## BadHubVersion

<sup>Severity: Critical</sup>

FinOps hubs 0.2 is not operational. Please upgrade to version 0.3 or later.

**Mitigation**: Upgrade to the latest version of [FinOps hubs](../_reporting/hubs/README.md).

<br>

## InvalidExportContainer

<sup>Severity: Critical</sup>

This file looks like it may be exported from Cost Management but it is not in the correct container.

**Mitigation**: Update your Cost Management export to point to the 'msexports' storage container. The 'ingestion' container is only used for querying ingested cost data.

<br>

## InvalidExportVersion

<sup>Severity: Critical</sup>

FinOps hubs requires FOCUS cost exports but this file looks like a legacy Cost Management export.

**Mitigation**: Create a new Cost Management export for FOCUS cost and either stop the current export or change it to export to a different storage container.

<br>

## InvalidHubVersion

<sup>Severity: Critical</sup>

FinOps hubs 0.1.1 and earlier do not work with the [Data ingestion Power BI report](../_reporting/power-bi/data-ingestion.md).

**Mitigation**: Upgrade to the latest version of [FinOps hubs](../_reporting/hubs/README.md) or download Power BI reports from [release 0.1.1](https://github.com/microsoft/finops-toolkit/releases/tag/v0.1.1).

<br>

## InvalidScopeId

<sup>Severity: Informational</sup>

The export path is not a valid scope ID. FinOps hubs expects the export path to be an Azure resource ID for the scope the export was created to simplify management. This shouldn't cause failures, but may result in confusing results for scope-related reports."

**Mitigation**: Update the storage path for the Cost Management export to use the full Azure resource ID for the scope.

<br>

## ExportNotFound

<sup>Severity: Critical</sup>

Cost Management data has not been exported.

**Mitigation**: Create a new FOCUS export in [Cost Management](https://aka.ms/exportsv2). After created, select 'Run now' to start the export process.

<br>

## NoDataIngested

<sup>Severity: Critical</sup>

The export path is not a valid scope ID. FinOps hubs expects the export path to be an Azure resource ID for the scope the export was created to simplify management. This shouldn't cause failures, but may result in confusing results for scope-related reports.

**Mitigation**: Update the storage path for the Cost Management export to use the full Azure resource ID for the scope.

<br>

## ManifestReadFailed

<sup>Severity: Critical</sup>

FinOps hub **msexports_ExecuteETL** pipeline failed to read the Cost Management manifest file.

**Mitigation**:

1. If the error occurred on a working hub instance when no changes were made to the hub or export, then Cost Management may have changed the manifest schema for an existing API version.
2. If the error occurred after creating a new or changing an existing export, then the export API version may use a new manifest schema that is not supported yet.
3. If the error occurred after a hub deployment (initial install or upgrade), then the deployment may have failed or there could be a bug in the pipeline.

To confirm the manifest schema (\#1) or API version (\#2):

1. Open the hub storage account in the Azure portal or storage explorer.
2. If in the Azure portal, go to **Storage browser** in the menu.
3. Select the **msexports** container.
4. Navigate down the file hierarchy for the export with the issue (see the manifest location in the error message).
5. Find the **manifest.json** file and select the menu (**⋯**) on the right, then select **View/edit**.
6. Identify the following properties:
   ```json
   {
     "exportConfig": {
       "resourceId": "<scope-id>/providers/Microsoft.CostManagement/exports/<export-name>",
       "dataVersion": "<dataset-version>",
       "apiVersion": "2023-07-01-preview",
       "type": "<dataset-type>",
       ...
     },
     ...
   }
   ```
7. Confirm the are set to the following supported values:
   - **resourceId** can be any scope ID and any export name, but it must exist with the "Microsoft.CostManagement/exports" resource type. This is case-insensitive.
   - **type** must exist, but should not fail with this error for any non-null value.
   - **dataVersion** must exist, but should not fail with this error for any non-null value.
   - **apiVersion** is not used explicitly but can signify changes to the manifest schema. See [supported API versions](../_reporting/hubs/data-processing.md#datasets) for details.
8. If you are using a newer API version:
   1. [Create a change request issue in GitHub](https://aka.ms/ftk/idea) to track adding support for the new API version.
   2. Delete the export in Cost Management.
   3. Create an export using the [New-FinOpsCostExport PowerShell command](../_automation/powershell/cost/New-FinOpsCostExport.md) using a supported API version.
   <blockquote class="tip" markdown="1">
     _If you consider yourself a power user, you may want to try updating the pipeline yourself for the quickest resolution. To do that, open Data Factory, navigate to Author > Pipelines > msexports_ExecuteETL, and select the applicable "Set" activities and update the **Settings** > **Value** property as needed. If you do this, you do not need to re-create the export with an older version. Please still report the issue and consider sharing the new JSON from the `{}` icon at the top-right of the pipeline designer._
   </blockquote>
9. If you notice the properties have changed for a supported API version:
   1. [Create a change request issue in GitHub](https://aka.ms/ftk/idea) to track the breaking change. Please include the **type**, **dataVersion**, and **apiVersion** from your manifest.json file.
   2. File a support request with Cost Management to request their change be reverted as it will break everyone using FinOps hubs or other custom solutions. Include the following details to help the Cost Management support team identify the issue within their system. Note Cost Management does not have context on FinOps hubs, so we should keep the details focused on Cost Management functionality.
      > I am using Cost Management exports to pull my cost data into ADLS. I have an ADF pipeline that is processing the data when manifest files are written. My pipeline was built on API version **<your-supported-api-version>** which expects `exportConfig.resourceId`, `exportConfig.type`, and `exportConfig.dataVersion` properties to be delivered consistently. I noticed these files are not being included in the manifest file for this API version for my export that ran on **<your-export-date>**. My expectation is that the manifest file should never change for an existing API version. Can you please revert these changes?
      >
      > To help you troubleshoot, here is my manifest file: <your-manifest-json>

If the manifest properties look good an this was a new or upgraded FinOps hub instance, confirm the deployment:

1. Open the hub resource group in the Azure portal.
2. Select **Settings** > **Deployments** in the menu on the left.
3. Confirm all deployments are successful. Specifically, look for the following deployment names:
   - main
   - hub
   - dataFactoryResources
   - storage
   - keyVault
4. If any deployments failed, review the error message to determine if it's something you can resolve yourself (e.g., name conflict, fixable policy violation).
5. If the error seems transient, try deploying again.
6. If the error persists, create a [discussion](https://aka.ms/ftk/discuss) to see if anyone else if facing an issue or knows of a possible workaround (especially for policy issues).
7. If the error is very clearly a bug or feature gap, [create a bug or feature request issue in GitHub](https://aka.ms/ftk/idea).

We try to respond to issues and discussions within 2 business days.

<!--
TODO: Consider the following ways to streamline this in the future:
1. Opt-in telemetry/email to the FTK team when errors happen in the pipeline
2. Detect these errors from the Data ingestion report.
3. Create a hub configuration workbook to detect configuration issues.
4. Consider renaming the main deployment file so it doesn't risk conflicting with other deployments.
-->

<br>

## ResourceAccessForbiddenException

Power BI: Exception of type 'Microsoft.Mashup.Engine.Interface.ResourceAccessForbiddenException' was thrown

Indicates that the account loading data in Power BI does not have the [Storage Blob Data Reader role](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-reader). Grant this role to the account loading data in Power BI.

<br>

## RoleAssignmentUpdateNotPermitted

If you've deleted FinOps Hubs and are attempting to redeploy it with the same values, including the Managed Identity name, you might encounter the following known issue:

```json
"code": "RoleAssignmentUpdateNotPermitted",
"message": "Tenant ID, application ID, principal ID, and scope are not allowed to be updated."
```

To fix that issue you will have to remove the stale identity:

- Navigate to the storage account and select **Access control (IAM)** in the menu.
- Select the **Role assignments** tab.
- Find any role assignments with an "unknown" identity and delete them.

<br>

## SchemaLoadFailed

<sup>Severity: Critical</sup>

FinOps hub **msexports_ETL_ingestion** pipeline failed to load the schema file.

**Mitigation**: Review the error message to note the dataset type and version, which are formatted with an underscore (e.g., `<type>_<version>` or `FocusCost_1.0`). Confirm that the dataset and type are both supported by the deployed version of FinOps hubs. See [supported datasets](../_reporting/hubs/data-processing.md#datasets) for details.

<br>

## SchemaNotFound

<sup>Severity: Critical</sup>

FinOps hub **msexports_ExecuteETL** pipeline was not able to find the schema mapping file for the exported dataset.

**Mitigation**: Confirm the dataset type and version are supported. See [supported datasets](../_reporting/hubs/data-processing.md#datasets) for details. If the dataset is supported, confirm the hub version with the [Data ingestion report](../_reporting/power-bi/data-ingestion.md).

To add support for another dataset, create a custom mapping file and save it to `config/schemas/<dataset-type>_<dataset-version>.json`. The `<dataset-type>` `<dataset-version>` values much match what Cost Management uses. Use an existing schema file as a template to identify the datatype for each column. Keep in mind some datasets have different schemas for EA and MCA, which cannot be identified via these attributes and may cause an issue if you have both account types. We will add all datasets in a future release and account for the EA and MCA differences by aligning to FOCUS.

<br>

## UnknownExportFile

<sup>Severity: Informational</sup>

The file in hub storage does not look like it was exported from Cost Management. File will be ignored.

**Mitigation**: The **msexports** container is intended for Cost Management exports only. Please move other files in another storage container.

<br>

## UnknownHubVersion

<sup>Severity: Critical</sup>

Unable to identify the version of FinOps hubs from the settings file. Please verify settings are correct. FinOps hubs 0.1.1 and earlier does not work with this Power BI report.

**Mitigation**: Upgrade to the latest version of [FinOps hubs](../_reporting/hubs/README.md) or download Power BI reports from https://github.com/microsoft/finops-toolkit/releases/tag/v0.1.1

<br>

## UnsupportedExportType

<sup>Severity: Warning</sup>

The export manifest in hub storage indicates the export was for an unsupported dataset. Exported data will be reported as ingestion errors.

**Mitigation**: Create a new Cost Management export for FOCUS cost and either stop the current export or change it to export to a different storage container.

<br>

## The \<name> resource provider is not registered in subscription \<guid>

Open the subscription in the Azure portal, then select **Settings** > **Resource providers**, select the resource provider row (e.g., Microsoft.EventGrid), then select the **Register** command at the top of the page. Registration may take a few minutes.

<br>

## x_PricingSubcategory shows the commitment discount ID

Cost Management exports before Feb 28, 2024 had a bug where `x_PricingSubcategory` was being set incorrectly for committed usage. You should expect to see values like `Committed Spend` and `Committed Usage`. Instead, you may see values like:

- `Committed /providers/Microsoft.BillingBenefits/savingsPlanOrders/###/savingsPlans/###`
- `Committed /providers/Microsoft.Capacity/reservationOrders/###/reservations/###`

If you see these values, please re-export the cost data for that month. If you need to export data for an older month that is not available, please contact support to request the data be exported for you to resolve the data quality issue from the previous export runs.

<br>

## Power BI: Reports are empty (no data)

If you don't see any data in your Power BI or other reports or tools, try the following based on your data source:

1. If using the Cost Management connector in Power BI, check the `Billing Account ID` and `Number of Months` parameters to ensure they're set correctly. Keep in mind old billing accounts may not have data in recent months.
2. If using FinOps hubs, check the storage account to ensure data is populated in the **ingestion** container. You should see either a **providers** or **subscriptions** folder. Use the sections below to troubleshoot further.

### FinOps hubs: Ingestion container is empty

If the **ingestion** container is empty, open the Data Factory instance in Data Factory Studio and select **Manage** > **Author** > **Triggers** and verify the **msexports_FileAdded** trigger is started. If not, start it.

If the trigger fails to start with a "resource provider is not registered" error, open the subscription in the Azure portal, then select **Settings** > **Resource providers**, select the **Microsoft.EventGrid** row, then select the **Register** command at the top of the page. Registration may take a few minutes.

After registration completes, start the **msexports_FileAdded** trigger again.

After the trigger is started, re-run all connected Cost Management exports. Data should be fully ingested within 10-20 minutes, depending on the size of the account.

If the issue persists, check if Cost Management exports are configured with file partitioning enabled. If you find it disabled, turn it on and re-run the exports.

Confirm the **ingestion** container is populated and refresh your reports or other connected tools.

### FinOps hubs: Files available in the ingestion container

If the **ingestion** container is not empty, confirm whether you have **parquet** or **csv.gz** files by drilling into the folders.

Once you know, verify the **FileType** parameter is set to `.parquet` or `.gz` in the Power BI report. See [Connect to your data](../_reporting/power-bi/README.md#-connect-to-your-data) for details.

If you're using another tool, ensure it supports the file type you're using.

<br>

## Power BI: The remote name could not be resolved: '\<storage-account>.dfs.core.windows.net'

Indicates that the storage account name is incorrect. If using FinOps hubs, verify the **StorageUrl** parameter from the deployment. See [Connect to your data](../_reporting/power-bi/README.md#-connect-to-your-data) for details.

<br>

## Power BI: We cannot convert the value null to type Logical

Indicates that the **Billing Account ID** parameter is empty. If using FinOps hubs, set the value to the desired billing account ID. If you do not have access to the billing account or do not want to include commitment purchases and refunds, set the value to `0` and open the **CostDetails** query in the advanced editor and change the `2` to a `1`. This will inform the report to not load actual/billed cost data from the Cost Management connector. See [Connect to your data](../_reporting/power-bi/README.md#-connect-to-your-data) for details.

Applicable versions: **0.1 - 0.1.1** (fixed in **0.2**)

<br>

## FinOps hubs: We cannot convert the value null to type Table

This error typically indicates that data was not ingested into the **ingestion** container.

If you just upgraded to FinOps hubs 0.2, this may be due to the Power BI report being old (from 0.1.x) or because you are not using FOCUS exports. See the [Upgrade guide](../_reporting/hubs/upgrade.md) for details.

See [Reports are empty (no data)](#power-bi-reports-are-empty-no-data) for additional troubleshooting steps.

<br>

## Next steps

Didn't find what you're looking for?

[Start a discussion](https://aka.ms/finops/toolkit/discuss){: .btn .btn-primary .mb-4 .mb-md-0 .mr-4 }
[Create an issue](https://aka.ms/ftk/idea){: .btn .mt-2 .mb-4 .mb-md-0 .mr-4 }

<br>
