---
title: "Resumable add table constraints"
description: New resumable capabilities to support pausing and resuming a running ALTER TABLE ADD CONSTRAINT operation. 
ms.custom:
- event-tier1-build-2022
ms.date: "05/24/2022"
ms.prod: sql
ms.reviewer: ""
ms.technology: security
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
monikerRange: ">=sql-server-ver16||>= sql-server-linux-ver16"
---

# Resumable add table constraints

[!INCLUDE [SQL Server 2022](../../includes/applies-to-version/sqlserver2022.md)]

Starting with SQL Server 2022, resumable operations for online index creation and rebuild are supported. The resumable operations allow index operations to be executed while the table is [online](/sql/t-sql/statements/alter-table-transact-sql#with--online--on--off-as-applies-to-altering-a-column) (`ONLINE=ON`) and also:

- Pause and restart an index create or rebuild operation multiple times to fit a maintenance window

- Recover from index creation or rebuild failures, such as database failovers or running out of disk space.

- Enable truncation of transaction logs during an index creation or rebuild operation.

- When an index operation is paused, both the original index and the newly created one require disk space and need to be updated during [Data Manipulation Language (DML)](/sql/t-sql/statements/statements#data-manipulation-language) operations.

The new SQL Server extensions allow a resumable operation for the [Data Definition Language (DDL)](/sql/t-sql/statements/statements#data-definition-language) command [ALTER TABLE ADD CONSTRAINT](/sql/t-sql/statements/alter-table-transact-sql) and adding a Primary or Unique Key. For more information on adding a Primary or Unique Key, see [ALTER TABLE table_constraint](/sql/t-sql/statements/alter-table-table-constraint-transact-sql).

## Resumable operations

In previous versions of SQL Server, the `ALTER TABLE ADD CONSTRAINT` operation can be executed with the `ONLNE=ON` option. However, the operation may take many hours for a large table to complete, and can consume a great number of resources. There's also the possibility of failures or interruption during such execution. We've introduced resumable capabilities to `ALTER TABLE ADD CONSTRAINT` for users to pause the operation during a maintenance window, or to restart it from where it was interrupted during an execution failure, without restarting the operation from the beginning.

## Supported scenarios

The new resumable capability for `ALTER TABLE ADD CONSTRAINT` supports the following customer scenarios:

- Pause or resume running `ALTER TABLE ADD CONSTRAINT` operation, such as pausing it for a maintenance window, and resuming the operation once the maintenance window is complete.

- Resume `ALTER TABLE ADD CONSTRAINT` operation after failovers and system failures.

- Executing `ALTER TABLE ADD CONSTRAINT` operation on a large table despite the small log size available.

> [!NOTE]
> The resumable operation for `ALTER TABLE ADD CONSTRAINT` requires the `ALTER` command to be executed online (`WITH ONLINE = ON`).

This feature is especially useful for large tables.

## T-SQL Syntax for ALTER TABLE

For information on the syntax used to enable resumable operations on a table constraint, see the syntax and options in [ALTER TABLE (Transact-SQL)](/sql/t-sql/statements/alter-table-transact-sql#resumable).

### Remarks for ALTER TABLE

- A new clause **WITH <resumable_options** have been added to the current T-SQL syntax in [ALTER TABLE (Transact-SQL)](/sql/t-sql/statements/alter-table-transact-sql).

- The option **RESUMABLE** is new and has been added to the existing [ALTER TABLE (Transact-SQL)](/sql/t-sql/statements/alter-table-transact-sql) syntax.

- `MAX_DURATION` = *time* \[MINUTES\] used with `RESUMABLE = ON` (requires `ONLINE = ON`). `MAX_DURATION` indicates time (an integer value specified in minutes) that a resumable online add constraint operation is executed before being paused. If not specified, the operation continues until completion.

## T-SQL syntax for ALTER INDEX

To pause, resume, or abort the resumable table constraint operation for `ALTER TABLE ADD CONSTRAINT`, use the T-SQL syntax [ALTER INDEX (Transact-SQL)](/sql/t-sql/statements/alter-index-transact-sql).

For resumable constraints the existing ALTER INDEX ALL command is used.

```syntaxsql
ALTER INDEX ALL ON <table_name>  
      { RESUME [WITH (<resumable_index_options>,[...n])]
        | PAUSE
        | ABORT
      }
<resumable_index_option> ::=
 { 
    MAXDOP = max_degree_of_parallelism
    | MAX_DURATION =<time> [MINUTES]
    | <low_priority_lock_wait>  
 }
 <low_priority_lock_wait>::=  
{  
    WAIT_AT_LOW_PRIORITY ( MAX_DURATION = <time> [ MINUTES ] ,   
                          ABORT_AFTER_WAIT = { NONE | SELF | BLOCKERS } )  
}  
```

### Remarks for ALTER INDEX

`ALTER INDEX ALL ON <Table> PAUSE`

- Pause a resumable and online add table constraint operation that is being executed

`ALTER INDEX ALL ON <Table> RESUME [WITH (<resumable_index_options>,[...n])]`

- Resume an add table constraint operation that is paused manually or due to a failure.

`MAX_DURATION` used with `RESUMABLE=ON `

- The time (an integer value specified in minutes) that the resumable add table constraint operation is executed after being resumed. Once the time expires, the resumable operation is paused if it's still running.

`WAIT_AT_LOW_PRIORITY` used with `RESUMABLE=ON` and `ONLINE = ON`

- Resuming an online add table constraint operation after a pause must wait for blocking operations on this table. `WAIT_AT_LOW_PRIORITY` indicates that the add table constraint operation will wait for low priority locks, allowing other operations to proceed while the resumable operation is waiting. Omitting the `WAIT_AT_LOW_PRIORITY` option is equivalent to `WAIT_AT_LOW_PRIORITY (MAX_DURATION = 0 minutes, ABORT_AFTER_WAIT = NONE)`. For more information, see [WAIT_AT_LOW_PRIORITY](/sql/t-sql/statements/alter-index-transact-sql#wait-at-low-priority).

`ALTER INDEX ALL ON <Table> ABORT`

- Abort a running or paused add table constraint operation that was declared as resumable. The abort operation must be explicitly executed as an `ABORT` command to terminate a resumable constraint operation. Failure or pausing a resumable table constraint operation doesn't terminate its execution. Rather, it leaves the operation in an indefinite paused state.

For more information on `PAUSE`, `RESUME`, and `ABORT` options available for resumable operations, see [ALTER INDEX (Transact-SQL)](/sql/t-sql/statements/alter-index-transact-sql).

## View the status for resumable operation

To view the status for the resumable table constraint operation, use the view [sys.index_resumable_operations](/sql/relational-databases/system-catalog-views/sys-index-resumable-operations).

## Permissions

Requires `ALTER` permission on the table.

No new permissions for resumable `ALTER TABLE ADD CONSTRAINT` are required.

## Examples

Here are some examples on using resumable add table constraint operations.

### Example 1

Resumable `ALTER TABLE` operation for adding a primary key clustered on column (a) with `MAX_DURATION` of 240 minutes.

```sql
ALTER TABLE table1
ADD CONSTRAINT PK_Constrain PRIMARY KEY CLUSTERED (a)
WITH (ONLINE = ON, MAXDOP = 2, RESUMABLE = ON, MAX_DURATION = 240);
```

### Example 2

Resumable `ALTER TABLE` operation for adding a unique constraint on two columns (a and b) with `MAX_DURATION` of 240 minutes.

```sql
ALTER TABLE table2
ADD CONSTRAINT PK_Constrain UNIQUE CLUSTERED (a,b)
WITH (ONLINE = ON, MAXDOP = 2, RESUMABLE = ON, MAX_DURATION = 240);
```

### Example 3

`ALTER TABLE` operation for adding a primary key clustered being paused and resumed.

The table below shows two sessions (`Session #1` and `Session #2`) being executed chronologically using the following T-SQL statements. `Session #1` executes a resumable `ALTER TABLE ADD CONSTRAINT` operation creating a primary key on column `Col1`. `Session #2` checks the execution status for the running constraint. After some time, it pauses the reusable operation. `Session #2` checks the status for the paused constraint. Finally, `Session #1` resumes the paused constraint and `Session #2` checks the status again.

|Session #1|Session #2|
|-----|-----|
|Execute resumable add constraint <br><br> `ALTER TABLE TestConstraint` <br> `ADD CONSTRAINT PK_TestConstraint PRIMARY KEY (Col1)` <br> `WITH (ONLINE = ON, MAXDOP = 2, RESUMABLE = ON, MAX_DURATION = 30);`||
||Check the constraint status <br><br> `SELECT sql_text, state_desc, percent_complete` <br> `FROM sys.index_resumable_operations;`|
|Output showing the operation <br><br> <table border="1"><tr><td>**sql_text**</td><td>**state_desc**</td><td>**percent_complete**</td></tr><tr><td>`ALTER TABLE TestConstraint (...)`</td><td>RUNNING</td><td>43.552</td></tr></table>||
||Pause the resumable constraint <br><br> `ALTER INDEX ALL ON TestConstraint PAUSE;`|
|**Error** <br><br> `Msg 1219, Level 16, State 1, Line 6` <br> `Your session has been disconnected because of a high priority DDL operation.` <br><br> `Msg 1750, Level 16, State 1, Line 6` <br> `Could not create constraint or index. See previous errors.` <br><br> `Msg 0, Level 20, State 0, Line 5` <br> `A severe error occurred on the current command.` <br> `The results, if any, should be discarded.`||
||Check the constraint status <br><br> `SELECT sql_text, state_desc, percent_complete` <br> `FROM sys.index_resumable_operations;`|
|Output showing the operation <br><br> <table border="1"><tr><td>**sql_text**</td><td>**state_desc**</td><td>**percent_complete**</td></tr><tr><td>`ALTER TABLE TestConstraint (...)`</td><td>PAUSED</td><td>65.339</td></tr></table>||
|`ALTER INDEX ALL ON TestConstraint RESUME;`||
||Check the constraint status <br><br> `SELECT sql_text, state_desc, percent_complete` <br> `FROM sys.index_resumable_operations;`|
|Output showing the operation <br><br> <table border="1"><tr><td>**sql_text**</td><td>**state_desc**</td><td>**percent_complete**</td></tr><tr><td>`ALTER TABLE TestConstraint (...)`</td><td>RUNNING</td><td>90.238</td></tr></table>||

Once the operation is completed, execute the following T-SQL statement to check the constraint:

```sql
SELECT constraint_name, table_name, constraint_type 
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS WHERE CONSTRAINT_TYPE='PRIMARY KEY';
GO
```

Here's the result set:

|constraint_name|table_name|constraint_type|
|-----|-----|-----|
|PK_Constraint|TestConstraint|PRIMARY KEY|

## See also

- [Guidelines for Online Index Operations](/sql/relational-databases/indexes/guidelines-for-online-index-operations)
- [CREATE INDEX (Transact-SQL)](/sql/t-sql/statements/create-index-transact-sql)
- [ALTER INDEX (Transact-SQL)](/sql/t-sql/statements/alter-index-transact-sql)
- [sys.index_resumable_operations](/sql/relational-databases/system-catalog-views/sys-index-resumable-operations)
- [WAIT_AT_LOW_PRIORITY](/sql/t-sql/statements/alter-index-transact-sql#wait-at-low-priority)
- [ALTER TABLE (Transact-SQL)](/sql/t-sql/statements/alter-table-transact-sql)
- [ALTER TABLE index_option](/sql/t-sql/statements/alter-table-index-option-transact-sql)
