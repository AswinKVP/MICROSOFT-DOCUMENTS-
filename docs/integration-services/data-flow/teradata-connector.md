---
description: "Microsoft connector for Teradata"
title: "Microsoft connector for Teradata | Microsoft Docs"
ms.custom: ""
ms.date: "11/22/2019"
ms.prod: sql
ms.prod_service: "integration-services"
ms.reviewer: ""
ms.technology: integration-services
ms.topic: conceptual
author: chugugrace
ms.author: chugu
---
# Microsoft connector for Teradata

[!INCLUDE[sqlserver-ssis](../../includes/applies-to-version/sqlserver-ssis.md)]

Microsoft connector for Teradata enables exporting data from and load data into Teradata databases in an SSIS package.

This new connector supports databases with 1MB-enabled tables.

## Version support

The following Microsoft SQL Server products are supported by Microsoft Connector for Teradata:

- Microsoft SQL Server 2019
- Microsoft SQL Server Data Tools (SSDT) 15.8.1 or later for Visual Studio 2017
- Microsoft SQL Server Data Tools (SSDT) for Visual Studio 2019

Microsoft connector for Teradata uses ODBC Driver for Teradata and Teradata Parallel Transporter shipped with Teradata Tools and Utilities (TTU). Supported TTU versions are 16.20 and 17.10.

## Installation

Install TTU from [Teradata site](https://downloads.teradata.com/download/tools/teradata-tools-and-utilities-windows-installation-package). Make sure that ODBC Driver for Teradata and Teradata Parallel Transporter features is selected during the installation.

To install the connector for Teradata database, download and run the installer from [the latest version of Microsoft connector for Teradata](https://www.microsoft.com/download/details.aspx?id=100599). Then follow the directions in the installation wizard.

After you install the connector, you must restart the SQL Server Integration Service to be sure that the Teradata source and destination works correctly.

## Design and execute SSIS packages

Microsoft Connector for Teradata provides similar user experience with Attunity Teradata Connector. User can design new packages based on previous experience, using SSDT for VS 2017 or VS 2019, with *targeting SQL server 2019*.

Teradata source and destination are under Common category.

![The Teradata Component](media/teradata-component.png)

Teradata connection manager is displayed as "TERADATA".

![The Teradata connection manager type](media/teradata-connection-manager-type.png)

Existing SSIS packages that have been designed with Attunity Teradata Connector will be automatically upgraded to use Microsoft Connector for Teradata. The icons will be changed as well.

To execute SSIS package *targeting SQL Server 2017 and below*, you will need to install **Microsoft Connector for Teradata by Attunity** with corresponding version from below link:

- [SQL Server 2017: Microsoft Connector Version 5.0 for Teradata by Attunity](https://www.microsoft.com/download/details.aspx?id=55179)
- [SQL Server 2016: Microsoft Connector Version 4.0 for Teradata by Attunity](https://www.microsoft.com/download/details.aspx?id=52950)
- [SQL Server 2014: Microsoft Connector Version 3.0 for Teradata by Attunity](https://www.microsoft.com/download/details.aspx?id=44582)
- [SQL Server 2012: Microsoft Connector Version 2.0 for Teradata by Attunity](https://www.microsoft.com/download/details.aspx?id=29283)

To design SSIS package in SSDT *targeting SQL Server 2017 and below*, you will need to have **Microsoft Connector for Teradata** and install **Microsoft Connector for Teradata by Attunity** with corresponding version.

## Limitations and known issues

- Teradata Source/Destination Editor, **Default database** property does not take effective. As work-around, type database name in dropdown box to filter table or view.

- Teradata Source/Destination Editor, Mapping step does not work when type \<database>.<table/view>. As work-around, type \<database>.<table/view>, then click the drop-down button.

- Teradata Source Editor, view cannot be displayed when Data access mode is "Table Name – TPT Export". As work-around, use Advanced Editor of Teradata Source.

- Teradata Destination, attribute "PackMaximum" cannot be set to "True". Otherwise, error will occur.

- Teradata Source always reads columns in the order as they are defined in the table. ODBC, on the other hand, requires columns be read in a specific order in certain circumstances [Getting Long Data](../../odbc/reference/develop-app/getting-long-data.md) and [SQLGetData](../../relational-databases/native-client-odbc-api/sqlgetdata.md). When the two orders do not match, read fails with error message "Invalid descriptor index, descriptor record does not exist, or descriptor record was not properly initialized."

- Installing a new version over an old installation will not remove the old version entry from the installed program list. While this does not impact functioning of the new version, user can manually uninstall the old version in control panel "Programs and Features" to get rid of the old version entry.

## Uninstallation

You can run uninstall wizard to remove **Microsoft connector for Teradata**.

## Release Notes

### Rev. 225

**New Features**
- Add support for TTU 17.10.

**Bugfixes**
- Incorrect data may be loaded by Teradata Source under certain circumstances.
- Default error table in Teradata Destination is incorrectly named and may not be created in the database of destination table.
- Options specified in the connection string property of Teradata Connection Manager may not be picked up and therefore do not take effect.

### Rev. 197

**Bugfixes**
- When reading empty string data, Teradata Source would fail with error message "An error occurred when converting string to target codepage."

## Next steps

- Configure [Teradata connection manager](teradata-connection-manager.md)
- Configure [Teradata source](teradata-source.md)
- Configure [Teradata destination](teradata-destination.md)
- If you have questions, visit [Tech Community](https://aka.ms/AA6iwdw).
