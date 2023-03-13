---
title: "MSSQLSERVER_17884"
description: "MSSQLSERVER_17884"
author: MashaMSFT
ms.author: mathoma
ms.date: "04/04/2017"
ms.service: sql
ms.subservice: supportability
ms.topic: "reference"
helpviewer_keywords:
  - "17884 (Database Engine error)"
---
# MSSQLSERVER_17884
 [!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]
  
## Details  
  
| Attribute | Value |  
| :-------- | :---- |  
|Product Name|SQL Server|  
|Event ID|17884|  
|Event Source|MSSQLSERVER|  
|Component|SQLEngine|  
|Symbolic Name|SRV_SCHEDULER_DEADLOCK|  
|Message Text|New queries assigned to process on Node %d have not been picked  up by a worker thread in the last %d seconds. Blocking or long-running queries can contribute to this condition, and may degrade client response time. Use the "max worker threads" configuration option to increase number  of allowable threads, or optimize current running queries.  SQL Process Utilization: %d%%. System Idle: %d%%.|  
  
## Explanation  
There is no sign of progress in each of the schedulers and could be caused by deadlocks where none of the threads can advance and/or no new work can be picked up and processed. If process utilization is low then other processes on the machine may be causing the server process CPU starvation.  
  
## User Action  
Determine why there is blocking and no progress being made and resolve situation accordingly. If process utilization is low check the load on the system caused by other processes.  
  
