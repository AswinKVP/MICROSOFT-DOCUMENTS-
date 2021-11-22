---
description: "MSSQLSERVER_701"
title: "MSSQLSERVER_701"
ms.custom: ""
ms.date: "11/04/2021"
ms.prod: sql
ms.technology: supportability
ms.topic: "reference"
helpviewer_keywords: 
  - "701 (Database Engine error)"
author: MashaMSFT
ms.author: mathoma
ms.reviewer: wiassaf
---
# MSSQLSERVER_701
 [!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]
  
## Details  
  
| Attribute | Value |  
| :-------- | :---- |  
|Product Name|SQL Server|  
|Event ID|701|  
|Event Source|MSSQLSERVER|  
|Component|SQLEngine|  
|Symbolic Name|NOSYSMEM|  
|Message Text|There is insufficient system memory to run this query.|  

> [!NOTE]
> **This article is focused on SQL Server.** For information on troubleshooting out of memory issues in Azure SQL Database, see [Troubleshoot out of memory errors with Azure SQL Database](/azure/azure-sql/database/troubleshoot-memory-errors-issues).

## Explanation  
This error occurs when [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] has failed to allocate sufficient memory to run the query. This can be caused by various reasons including operating system settings, physical memory availability, or memory limits on the current workload. In most cases, the transaction that failed is not the cause of this error.  
  
Diagnostic queries, such as DBCC statements, may fail because the server does not have sufficient memory.  
  
A timeout occurred while waiting for memory resources to execute the query in the resource pool 'default'.  
  
## User action  
If you are not using Resource Governor, we recommend that you verify the overall server state and load, or check the resource pool or workload group settings.  
  
The following list outlines general steps that will help in troubleshooting memory errors:  
  
1.  Verify whether other applications or services are consuming memory on this server. Reconfigure less critical applications or services to consume less memory.  
  
2.  Start collecting performance monitor counters for [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]**: Buffer Manager**, **SQL Server: Memory Manager**.  
  
3.  Check the following SQL Server memory configuration parameters:  
  
    -   **max server memory**  
  
    -   **min server memory**  
  
    -   **min memory per query**  
  
    Notice unusual settings. Correct them as necessary. Account for increased memory requirements. Default settings are listed in [Server memory configuration options](../../database-engine/configure-windows/server-memory-server-configuration-options.md).
  
4.  Observe DBCC MEMORYSTATUS output and the way it changes when you see these error messages.  
  
5.  Check the workload (for example, number of concurrent sessions, currently executing queries).  

The following actions may make more memory available to [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]:  
  
-   If applications besides [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] are consuming resources, try stopping running these applications or consider running them on a separate server. These steps will remove external memory pressure.  
  
-   If you have configured **max server memory**, increase its setting. For more information, see [Set options manually](../../database-engine/configure-windows/server-memory-server-configuration-options.md#manually).
  
Run the following DBCC commands to free several [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] memory caches.  
  
-   DBCC FREESYSTEMCACHE  
  
-   DBCC FREESESSIONCACHE  
  
-   DBCC FREEPROCCACHE  
  
If the problem continues, you will need to investigate further and possibly increase server resources or reduce workload.  
  
