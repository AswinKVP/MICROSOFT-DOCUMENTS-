---
title: "sys.sp_manage_distributed_transaction (Transact-SQL)"
description: "sys.sp_manage_distributed_transaction (Transact-SQL)"
author: VanMSFT
ms.author: vanto
ms.date: "02/24/2023"
ms.service: sql
ms.subservice: system-objects
ms.topic: "reference"
monikerRange: ">= sql-server-ver16||>= sql-server-linux-ver16||=azuresqldb-mi-current"
---
# sys.sp_manage_distributed_transaction (Transact-SQL)

[!INCLUDE [SQL Server 2022 Azure SQL Managed Instance](../../includes/applies-to-version/sqlserver2022-asmi.md)]

  Commits, aborts, or forgets a specified transaction  
  
 :::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)  
  
## Syntax  
  
```syntaxsql
sys.sp_manage_distributed_transaction 
   @transaction_uow = 'ID', 
   @operation = 'value'  
```  
  
## Arguments

`@transaction_uow = 'id'`
 Specifies MSDTC transaction ID GUID (transaction UOW).
  
`@operation = 'value'`
 Specifies operation to perform. Valid values are `commit`, `abort`, or `forget`.
  
## Return Code Values

**0** (success) or **1** (failure)  
  
## Result Sets

None  
  
## Permissions

Requires **sysadmin** or have **CONTROL SERVER** permissions.
  
## Examples
  
```sql
EXEC sys.sp_manage_distributed_transaction 
   @transaction_uow = '1101AD68-43A7-4DC5-B06C-2B4BEF230643', 
   @operation=N'commit' 
```  
  
## See also  

- [sys.dm_tran_distributed_transaction_stats (Transact-SQL)](../system-dynamic-management-views/sys-dm-tran-distributed-transaction-stats.md)
- [sp_reset_dtc_log (Transact-SQL)](sp-reset-dtc-log.md)
