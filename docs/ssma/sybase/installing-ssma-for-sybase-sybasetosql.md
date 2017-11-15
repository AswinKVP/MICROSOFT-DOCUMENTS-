---
title: "Installing SSMA for SAP ASE (SybaseToSQL) | Microsoft Docs"
ms.custom: ""
ms.date: "09/30/2017"
ms.prod: "sql-non-specified"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "sql-ssma"
ms.tgt_pltfrm: ""
ms.topic: "article"
applies_to: 
  - "Azure SQL Database"
  - "SQL Server"
ms.assetid: 8d5a4ce6-b747-46e3-9184-645d56e8b35c
caps.latest.revision: 6
author: "Shamikg"
ms.author: "Shamikg"
manager: "jhubbard"
ms.workload: "Inactive"
---
# Installing SSMA for SAP ASE (SybaseToSQL)
[!INCLUDE[msCoName](../../includes/msconame_md.md)][!INCLUDE[ssNoVersion](../../includes/ssnoversion_md.md)] Migration Assistant (SSMA) for SAP ASE consists of a client application that you use to perform a migration from SAP Adaptive Server Enterprise (ASE) to [!INCLUDE[ssNoVersion](../../includes/ssnoversion_md.md)] or Azure SQL Database. It also contains an extension pack that supports data migration and the use of ASE system functions in your migrated databases.  
  
You install the client application on the computer from which you will perform the migration steps. You must install the extension pack files on the computer that is running [!INCLUDE[ssNoVersion](../../includes/ssnoversion_md.md)] where migrated databases will be hosted.  
  
## Upgrading SSMA for Sybase  
If you want to upgrade to a later version of SSMA for SAP ASE, you must first uninstall the client and the server extension pack, and then install the newer version.  
  
If you open a project from an earlier version of SSMA for SAP ASE, SSMA asks if you want to convert the project to the newer version. You must click **Yes** to work with the project in the newer version of SSMA.  
  
## Contents  
  
|Topic|Description|  
|---------|---------------|  
|[Installing SSMA for SAP ASE Client &#40;SybaseToSQL&#41;](../../ssma/sybase/installing-ssma-for-sybase-client-sybasetosql.md)|Provides information about and instructions for installing the SSMA client.|  
|[Installing SSMA Components on SQL Server &#40;SybaseToSQL&#41;](../../ssma/sybase/installing-ssma-components-on-sql-server-sybasetosql.md)|Provides information about and instructions for installing the extension pack on instances of [!INCLUDE[ssNoVersion](../../includes/ssnoversion_md.md)].|  
|[Removing SSMA for SAP ASE Components &#40;SybaseToSQL&#41;](../../ssma/sybase/removing-ssma-for-sybase-components-sybasetosql.md)|Provides instructions for uninstalling the client program and extension pack.|  
  
## See also  
[Migrating SAP ASE databases to SQL Server - Azure SQL Database &#40;SybaseToSQL&#41;](../../ssma/sybase/migrating-sybase-ase-databases-to-sql-server-azure-sql-db-sybasetosql.md)  
