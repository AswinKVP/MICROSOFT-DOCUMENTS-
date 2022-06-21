---
title: "Digest management"
description: This article provides information on digest management for a ledger database.
ms.date: "05/24/2022"
ms.service: sql-database
ms.subservice: security
ms.custom:
- event-tier1-build-2022
ms.reviewer: kendralittle, mathoma
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
monikerRange: "= azuresqldb-current||>= sql-server-ver16||>= sql-server-linux-ver16"
---

# Digest management

[!INCLUDE [SQL Server 2022 Azure SQL Database](../../../includes/applies-to-version/sqlserver2022-asdb.md)]

## Database digests

The hash of the latest block in the database ledger is called the *database digest*. It represents the state of all ledger tables in the database at the time when the block was generated. Generating a database digest is efficient, because it involves computing only the hashes of the blocks that were recently appended. 

Database digests can be generated either automatically by the system or manually by the user. You can use them later to verify the integrity of the database. 

Database digests are generated in the form of a JSON document that contains the hash of the latest block, together with metadata for the block ID. The metadata includes the time that the digest was generated and the commit time stamp of the last transaction in this block.

The verification process and the integrity of the database depend on the integrity of the input digests. For this purpose, database digests that are extracted from the database need to be stored in trusted storage that the high-privileged users or attackers of the database can't tamper with.

### Automatic generation and storage of database digests

> [!NOTE]
> Automatic generation and storage of database digests is currently available in Azure SQL Database, but not supported on SQL Server.

Ledger integrates with the [immutable storage feature of Azure Blob Storage](/azure/storage/blobs/immutable-storage-overview) and [Azure Confidential Ledger](/azure/confidential-ledger/index). This integration provides secure storage services in Azure to help protect the database digests from potential tampering. This integration provides a simple and cost-effective way for users to automate digest management without having to worry about their availability and geographic replication.  Azure Confidential Ledger has a stronger integrity guarantee for customers who might be concerned about privileged administrators access to the digest. [This table](/azure/architecture/guide/technology-choices/multiparty-computing-service#confidential-ledger-and-azure-blob-storage) compares the immutable storage feature of Azure Blob Storage with Azure Confidential Ledger.  

You can configure automatic generation and storage of database digests through the Azure portal, PowerShell, or the Azure CLI. For more information, see [Enable automatic digest storage](ledger-how-to-enable-automatic-digest-storage.md). When you configure automatic generation and storage, database digests are generated on a predefined interval of 30 seconds and uploaded to the selected storage service. If no transactions occur in the system in the 30-second interval, a database digest won't be generated and uploaded. This mechanism ensures that database digests are generated only when data has been updated in your database. When the endpoint is an Azure Blob Storage, the database server will create a new container, named **sqldbledgerdigests** and use a naming pattern like:
ServerName/DatabaseName/CreationTime. The creation time is needed because a database with the same name can be dropped and recreated or restored, allowing for different “incarnations” of the database under the same name. See [Digest Management Considerations](ledger-digest-management.md)

> [!IMPORTANT]
> If you use Azure Blob Storage, configure an [immutability policy](/azure/storage/blobs/immutable-policy-configure-version-scope) on your container after provisioning to ensure that database digests are protected from tampering.

### Manual generation and storage of database digests

You can also generate a database digest on demand so that you can manually store the digest in any service or device that you consider a trusted storage destination. For example, you might choose an on-premises write once, read many (WORM) device as a destination. You manually generate a database digest by running the [sys.sp_generate_database_ledger_digest](/sql/relational-databases/system-stored-procedures/sys-sp-generate-database-ledger-digest-transact-sql) stored procedure in either [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) or [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).

> [!IMPORTANT]
> Generating database digests requires the **GENERATE LEDGER DIGEST** permission. For details on permissions related to ledger tables, see [Permissions](/sql/relational-databases/security/permissions-database-engine#asdbpermissions). 

```sql
EXECUTE sp_generate_database_ledger_digest;
```

The returned result set is a single row of data. It should be saved to the trusted storage location as a JSON document as follows:

```json
    {
        "database_name":  "ledgerdb",
        "block_id":  0,
        "hash":  "0xDC160697D823C51377F97020796486A59047EBDBF77C3E8F94EEE0FFF7B38A6A",
        "last_transaction_commit_time":  "2020-11-12T18:01:56.6200000",
        "digest_time":  "2020-11-12T18:39:27.7385724"
    }
```

## Digest management considerations
> [!NOTE]
> This section only applies to Azure SQL Database, and not SQL Server.

### Database restore

Restoring the database back to an earlier point in time, also known as [Point in Time Restore](/azure/azure-sql/database/recovery-using-backups#point-in-time-restore), is an operation frequently used when a mistake occurs and users need to quickly revert the state of the database back to an earlier point in time. When uploading the generated digests to Azure Storage or Azure Confidential Ledger, the *create time* of the database is captured that these digests map to. Every time the database is restored, it's tagged with a new *create time* and this technique allows us to store the digests across different “incarnations” of the database. Ledger preserves the information regarding when a restore operation occurred, allowing the verification process to use all the relevant digests across the various incarnations of the database. Additionally, users can inspect all digests for different create times to identify when the database was restored and how far back it was restored to. Since this data is written in immutable storage, this information will be protected as well.

### Active geo-replication

Replication across geographic regions is asynchronous for performance reasons and, thus, allows the secondary database to be slightly behind compared to the primary. In the event of a geographic failover, any latest data that hasn't yet been replicated is lost. Ledger will only issue database digests for data that has been replicated to geographic secondaries to guarantee that digests will never reference data that might be lost in case of a geographic failover. This only applies for automatic generation and storage of database digests.

Dropping the link between the primary and the secondaries when ledger digests are configured isn't supported. You should first disable the *Enable automatic digest storage* database setting, remove the synchronization between the primary and the secondary and re-enable the *Enable automatic digest storage* database setting.

## Next steps

- [Ledger overview](ledger-overview.md)
- [Enable automatic digest storage](ledger-how-to-enable-automatic-digest-storage.md)
- [sys.sp_generate_database_ledger_digest](/sql/relational-databases/system-stored-procedures/sys-sp-generate-database-ledger-digest-transact-sql)
