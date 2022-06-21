---
title: Release notes for (SSMS) 19 (Preview)
description: Release notes for SQL Server Management Studio (SSMS) 19 (Preview)
ms.prod: sql
ms.prod_service: sql-tools
ms.technology: ssms
ms.topic: conceptual
ms.custom:
- event-tier1-build-2022
author: erinstellato-ms
ms.author: erinstellato
ms.reviewer: maghan
ms.date: 05/26/2022
---

# Release notes for SQL Server Management Studio (SSMS) 19 (Preview)

[!INCLUDE [SQL Server ASDB, ASDBMI, ASDW ](../includes/applies-to-version/sql-asdb-asdbmi-asa.md)]

To view the latest general availability (GA) version of SSMS release notes, visit [SSMS Release Notes](release-notes-ssms.md).

This article provides details about updates, improvements, and bug fixes for the current and previous versions of SSMS.

[!INCLUDE[ssms-connect-aazure-ad](../includes/ssms-connect-azure-ad.md)]

## Current SSMS release

:::image type="icon" source="media/download-icon.png" border="false":::**[Download SQL Server Management Studio (SSMS) 19](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x409)**

SSMS 19 Preview 2 is the latest preview release of SSMS. If you need a previous version of SSMS, see [previous SSMS releases](release-notes-ssms.md#previous-ssms-releases).

### 19.0

- Release number: 19.0 Preview 2
- Build number: 16.0.19056.0
- Release date: May 24, 2022

[Chinese (Simplified)](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x804) | [Chinese (Traditional)](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x404) | [English (United States)](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x409) | [French](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x40c) | [German](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x407) | [Italian](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x410) | [Japanese](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x411) | [Korean](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x412) | [Portuguese (Brazil)](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x416) | [Russian](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x419) | [Spanish](https://go.microsoft.com/fwlink/?linkid=2195969&clcid=0x40a)

#### What's new in 19.0 Preview 2

| New Item | Details |
|----------|---------|
| Azure Data Studio installation integration | Installation of SSMS installs Azure Data Studio 1.36.2. |
| Always Encrypted | Added the ability to explicitly configure an attestation protocol in the “Connect To Server” dialog when using Always Encrypted with secure enclaves (column encryption). |
| Client Driver | Changed to Microsoft.Data.SqlClient. |
| Contained Always On Availability Group | Added support for Contained Always On Availability Groups. |
| Data Classification | Improvements to Data Classification user interface. |
| Ledger | Added support for Ledger feature Database ledger. For more information, visit [What is the database ledger?](/sql/relational-databases/security/ledger/ledger-database-ledger). |
| Query Execution or Results | Improved checks for open connections. |
| Query Tuning Advisor | Updated user interface for improved accessibility. |
| Showplan | The showplan XML schema has been updated. |

#### Bug fixes in 19.0 Preview 2

> [!Note]
> Fixes from SSMS 18.11.1 and the next SSMS release are included in SSMS 19.0 Preview 2.

| New Item | Details |
|----------|---------|
| Availability Group Dashboard | Fixed the issue when connecting to the Availability Group Dashboard for an AG on SQL Server 2016, which resulted in "unknown property ClusterType" error. |
| Query Editor | Fixed issue with audible notification occurring when closing a query window. See [SSMS 18.11.1 Beeps When I Close a Query Window](/answers/questions/775502/ssms-18111-beeps-when-i-close-a-query-window.html). |

#### Known issues 19.0 Preview 2

| New Item | Details | Workaround |
|----------|---------|------------|
| Analysis Services | Connection to Analysis Services isn't available. | This will be available in a later preview, use SSMS 18.11.1 to connect to Analysis Services. |
| Azure SQL DB | Limitation with MSAL caching, which may require reauthentication when signing into Azure. | Reauthenticate if prompted. |
| Copy Database Wizard | Copying a database generates a log provider type error | Copy the database using an alternative method such as backup and restore to a new database. |
| Database Designer | Clicking the Design option for a view that references a table using spatial data causes SSMS to crash. | Use T-SQL to make changes to the view. |
| Ledger | Importing a bacpac or dacpac created from a database with the LEDGER = ON option, into a new database on-premises, fails due to the LEDGER property not being set. | Use backup and restore to create a new database on-premises with the LEDGER property enabled. |
| Maintenance Plan | The Maintenance Plan node isn't available in Object Explorer. | This will be available in a later preview, use SSMS 18.11.1 to view or edit Maintenance Plans. |
| SSMS Installer | The original installation of Japanese SSMS was not fully localized and removed from the download page. | This is fixed via an updated build, 16.0.19058.0, now available for download. |
| Storage Account | Trying to delete a container from a storage account fails with a (400) Bad Request error. | Use the Azure portal for container deletion. |
| Stretch DB | Removed Stretch DB Wizard. | Use T-SQL or an earlier version of SSMS (18.9.1 or below) to use the Stretch DB Wizard. |

You can reference [SQL Server user feedback](https://feedback.azure.com/forums/908035-sql-server) for other known issues and to provide feedback to the product team.

## Next steps

For a list of all SQL Server Management Studio downloads, search the [Microsoft Download Center](https://www.microsoft.com/download/search.aspx?q=sql%20server%20management%20studio&p=0&r=10&t=&s=Relevancy~Descending).  
  
For the latest preview of SQL Server Management Studio 19.0, see [Download SQL Server Management Studio &#40;SSMS&#41;](../ssms/download-sql-server-management-studio-ssms-19.md).
