---
title: "Change Tracking stored procedures (Transact-SQL)"
description: "Change Tracking stored procedures (Transact-SQL)"
author: JetterMcTedder
ms.author: bspendolini
ms.date: "10/21/2022"
ms.service: sql
ms.subservice: system-objects
ms.topic: "reference"
helpviewer_keywords:
  - "system stored procedures [SQL Server], change tracking"
  - "change tracking [SQL Server], stored procedures"
dev_langs:
  - "TSQL"
---

# Change Tracking stored procedures (Transact-SQL)

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

Change Tracking is a lightweight solution that provides an efficient data change tracking mechanism for applications, ETL processes, event capture, and auditing. This allows for quick and simple detection of changed data without the need for expensive and complex custom solutions traditionally involving a combination of triggers, timestamp columns, new tables to store tracking information, and cleanup processes.

## Stored procedures

- [sys.sp_flush_commit_table &#40;Transact-SQL&#41;](../../relational-databases/system-stored-procedures/sys-sp-flush-commit-table-transact-sql.md)
- [sys.sp_flush_commit_table_on_demand &#40;Transact-SQL&#41;](../../relational-databases/system-stored-procedures/sys-sp-flush-commit-table-on-demand-transact-sql.md)
- [sys.sp_flush_CT_internal_table_on_demand &#40;Transact-SQL&#41;](../../relational-databases/system-stored-procedures/sys-sp-flush-ct-internal-table-on-demand-transact-sql.md)

## See also

 [About Change Tracking &#40;Transact-SQL&#41;](../../relational-databases/track-changes/about-change-tracking-sql-server.md)  
 [Change Tracking Cleanup and Troubleshooting &#40;Transact-SQL&#41;](../../relational-databases/track-changes/cleanup-and-troubleshoot-change-tracking-sql-server.md)  
 [Change Tracking Functions &#40;Transact-SQL&#41;](../../relational-databases/system-functions/change-tracking-functions-transact-sql.md)  
 [Change Tracking System Tables &#40;Transact-SQL&#41;](../../relational-databases/system-tables/change-tracking-tables-transact-sql.md)  
