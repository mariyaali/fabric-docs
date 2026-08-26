---
title: Correlate Fabric capacity metrics with Warehouse query insights
description: Configure a Power BI report that correlates Fabric capacity utilization with Query Insights from customer-owned Warehouses.
author: mariyaali
ms.author: mariyaali
ms.reviewer: wiassaf
ms.service: fabric
ms.subservice: data-warehouse
ms.topic: how-to
ms.date: 08/25/2026
---

# Correlate Fabric capacity metrics with Warehouse query insights

[!INCLUDE [applies-to-version](includes/applies-to-version/fabric-dw.md)]

The Microsoft Fabric Capacity Metrics app helps you identify capacity
consumption, spikes, throttling, and item-level usage. Warehouse Query Insights
helps you understand which SQL statements ran and how they performed.

The Fabric DW Query Capacity Correlation report brings these two views together
in a customer-owned Power BI report. When capacity consumption increases, you
can use the report to find the Warehouse queries that were active during the
same period and review their duration, allocated CPU, scan volume, status,
user, application, and SQL text.

The report doesn't deploy objects to your Warehouses or store reusable
credentials.

Download the report from the
[Fabric DW Query Capacity Correlation GitHub repository](https://github.com/mariyaali/fabric-dw-query-capacity-correlation).

## Why use this report

A capacity spike tells you that resource consumption increased, but you might
still need to determine:

- Which Warehouse queries were active at that time?
- Which users or applications submitted those queries?
- Are recurring query patterns driving capacity consumption?
- Should you tune or reschedule a workload before increasing capacity?

Without a shared view, you must compare timestamps manually across Capacity
Metrics and Query Insights. This report places both datasets on the same time
axis, helping you investigate performance faster, prioritize workload
optimization, validate changes, and prepare evidence for support cases.

> [!NOTE]
> Time correlation identifies queries that were active during a capacity
> event. Review all available evidence before attributing the event to a
> specific query.

## Prerequisites

Before you begin, make sure you have:

- Power BI Desktop with Power BI project (PBIP) support.
- Windows PowerShell 5.1 or PowerShell 7.
- Azure CLI.
- Access to the **Fabric Capacity Metrics** semantic model.
- Read and Query Insights access to each Warehouse you want to include.

## Download the report

1. Open the [public GitHub repository](https://github.com/mariyaali/fabric-dw-query-capacity-correlation).
2. Clone the repository.
4. Open PowerShell in your local copy.
5. Unblock the configuration script:

   ```powershell
   Unblock-File .\Configure-CustomerTemplate.ps1
   ```

## Find the required values

You need your Capacity ID and the workspace that contains the **Fabric Capacity
Metrics** semantic model.

### Capacity ID

1. In Fabric, select **Settings** > **Admin portal**.
2. Open **Capacity settings** and select your capacity.
3. Copy the GUID after `/capacities/` in the browser URL.

### Capacity Metrics workspace

1. Open **OneLake catalog** in Fabric.
2. Filter to **Semantic model** and search for **Fabric Capacity Metrics**.
3. Copy the exact value in the **Workspace** column.

The workspace name typically resembles:

```text
Microsoft Fabric Capacity Metrics <installation date and time>
```

If you can't see the semantic model, ask the person who installed the Fabric
Capacity Metrics app for access or for the workspace name. Don't use the name
of a workspace that only contains Warehouses.

## Configure the report

1. Sign in to Azure CLI. If necessary, specify the tenant that owns your Fabric
   resources:

   ```azurecli
   az login --tenant 00000000-0000-0000-0000-000000000000
   az account show
   ```

2. Run the configuration script with your Capacity ID and Capacity Metrics
   workspace name:

   ```powershell
   .\Configure-CustomerTemplate.ps1 `
     -CapacityId "00000000-0000-0000-0000-000000000000" `
     -CapacityMetricsWorkspace "Microsoft Fabric Capacity Metrics <installation>"
   ```

The script discovers Warehouses that your account can access on the selected
capacity and creates a `Configured` folder. It builds the Capacity Metrics
connection automatically.

To discover every workspace on the capacity, use a Fabric administrator or
service principal with tenant read permissions. Otherwise, the script includes
only workspaces available to your signed-in account.

## Open and refresh the report

1. Open `Configured\Query Capacity Correlation.pbip` in Power BI Desktop.
2. Select **Transform data** > **Data source settings**.
3. Sign in to Capacity Metrics and each listed Warehouse SQL endpoint.
4. Set every data source's privacy level to **Organizational**.
5. Review and approve each native Query Insights prompt.
6. Refresh the semantic model.

:::image type="content" source="media/correlate-capacity-metrics-query-insights/capacity-usage-report-timeline-queries.gif" alt-text="Screenshot of Power BI report showing capacity utilization timeline, hourly query counts, and warehouse query details.":::

Power BI stores your credentials separately from the PBIP files.

## Investigate a capacity spike

1. Select an hour or time range with elevated capacity utilization.
2. Filter by Warehouse, artifact kind, query status, statement type, user, or
   application.
3. Compare query duration, allocated CPU, data scanned, and status.
4. Use **Distributed Statement ID** to correlate Query Insights with Capacity
   Metrics when the value is available.
5. Review the SQL text and query hash before tuning or rescheduling a workload.

Query Insights retains 30 days of history, excludes system queries, and might
take up to 15 minutes to show a completed query.

## Publish and configure refresh

1. Publish the report and semantic model to your Fabric workspace.
2. Open the semantic model settings.
3. Under **Gateway and cloud connections**, configure every Warehouse SQL
   source and the Capacity Metrics source.
4. Run an on-demand refresh and confirm that it succeeds before sharing the
   report.

Credentials entered in Power BI Desktop aren't transferred to the Fabric
service.

## Troubleshoot

### Capacity Metrics model isn't found

`PowerBIEntityNotFound` means the supplied workspace doesn't contain the
**Fabric Capacity Metrics** semantic model. Find the model in OneLake catalog
and rerun the script with its exact **Workspace** value.

### Analysis server isn't found

Confirm that the Capacity Metrics workspace name is exact. Then clear the
failed permission under **File** > **Options and settings** > **Data source
settings** and sign in again.

The configuration script accepts the workspace display name, not an XMLA
endpoint.

### Warehouses or queries are missing

Confirm that your account can access the Warehouse and query
`queryinsights.exec_requests_history`. If your account doesn't have
tenant-wide read permissions, the script discovers only workspaces that you
can access.

## Related content

- [What is the Microsoft Fabric Capacity Metrics app?](../enterprise/metrics-app.md)
- [Install the Microsoft Fabric Capacity Metrics app](../enterprise/metrics-app-install.md)
- [Query Insights](query-insights.md)
- [Power BI Desktop projects](https://learn.microsoft.com/power-bi/developer/projects/projects-overview)
