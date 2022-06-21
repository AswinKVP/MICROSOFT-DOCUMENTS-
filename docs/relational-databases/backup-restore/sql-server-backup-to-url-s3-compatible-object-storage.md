---
title: "SQL Server backup to URL for S3-compatible object storage"
description: Learn about the concepts, requirements, and components necessary for SQL Server to use the S3-compatible object storage as a backup destination.
ms.custom:
- event-tier1-build-2022
ms.date: 05/09/2022
ms.prod: sql
ms.prod_service: backup-restore
ms.reviewer: ""
ms.technology: backup-restore
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
monikerRange: ">=sql-server-ver16||>=sql-server-linux-ver16"
---
# SQL Server backup to URL for S3-compatible object storage

[!INCLUDE [SQL Server 2022](../../includes/applies-to-version/sqlserver2022.md)]

This article introduces the concepts, requirements and components necessary to [use S3-compatible object storage as a backup destination](sql-server-backup-and-restore-with-s3-compatible-object-storage.md). The backup and restore functionality is conceptually similar to working with [SQL Server backup to URL for Azure Blob Storage](sql-server-backup-to-url.md) as a backup device type.

For information on supported platforms, see [providers of S3-compatible object storage](sql-server-backup-and-restore-with-s3-compatible-object-storage.md#providers-of-s3-compatible-object-storage).

> [!NOTE]
> SQL Server backup and restore with S3-compatible object storage is in preview as a feature of SQL Server 2022.

## Overview

[!INCLUDE[sssql22-md](../../includes/sssql22-md.md)] introduces object storage integration to the data platform, enabling you to integrate SQL Server with S3 compatible object storage in addition to Azure Storage. To provide this integration SQL Server has been enhanced with a new S3 connector, which uses the S3 REST API to connect to any provider of S3-compatible object storage. [!INCLUDE[sssql22-md](../../includes/sssql22-md.md)] extends the existing BACKUP/RESTORE TO/FROM URL syntax by adding support for the new S3 connector using the REST API. 

URLs pointing to S3-compatible resources are prefixed with `s3://` to denote that the S3 connector is being used. URLs beginning with `s3://` will always assume that the underlying protocol will be `https`.

## Part numbers and file size limitations

To storage data the S3-compatible object storage provider must split files in multiple blocks called parts, similarly to [Azure block blobs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) when using Azure Blob Storage.

Each file can be split up to 10,000 parts, each part size will range from 5 MB to 20 MB, this range is controlled by the T-SQL BACKUP command through the parameter `MAXTRANSFERSIZE`. The default value of `MAXTRANSFERSIZE` is 10 MB, therefore the default size of each part is 10 MB.

The maximum supported size of a single file is the result of *10,000 parts \* `MAXTRANSFERSIZE`*, if it is required to backup a bigger file it must split/striped up to 64 URLs. The final maximum supported size of a file is *10,000 parts \* `MAXTRANSFERSIZE` \* URLs*.

> [!NOTE]
> The use of COMPRESSION is required in order to change MAXTRANSFERSIZE values

## Prerequisites for the S3 endpoint

The S3 endpoint must have been configured as follows:

- TLS must be configured. It is assumed that all connections will be securely transmitted over HTTPS not HTTP. The endpoint will be validated by a certificate installed on the SQL Server OS Host.
- A user (`Access Key ID`) has been configured and the secret (`Secret Key ID`) and that user is known to you. You will need both to authenticate against the S3 endpoint.
- At least one bucket has been configured. Buckets cannot be created or configured from [!INCLUDE[sssql22-md](../../includes/sssql22-md.md)].

## Security

### Backup Permissions

The following are security considerations and requirements when backing up to or restoring using an S3-compatible object storage:

- The user account that is used to issue BACKUP or RESTORE commands should be in the **db_backupoperator** database role with **Alter any credential** permissions.

In order for the user to write the contents of an S3 bucket the user will need to be allowed to perform the following actions against the S3 endpoint:

- ListBucket
- writeonly

### Restore Permissions

If the database being restored does not exist, the user must have `CREATE DATABASE` permissions to be able to execute RESTORE. If the database exists, RESTORE permissions default to members of the `sysadmin` and `dbcreator` fixed server roles and the owner (`dbo`) of the database.

RESTORE permissions are given to roles in which membership information is always readily available to the server. Because fixed database role membership can be checked only when the database is accessible and undamaged, which is not always the case when RESTORE is executed, members of the `db_owner` fixed database role do not have RESTORE permissions.

In order for the user to read the contents of an S3 bucket the user will need to be allowed to perform the following actions against the S3 endpoint:

- ListBucket
- readonly

## Supported features

High-level overview of the supported features for `BACKUP` and `RESTORE`

1. A single backup file can be up to 200,000 MiB per URL (with `MAXTRANSFERSIZE` set to 20 MB).
2. Backups can be striped across a maximum of 64 URLs.
3. Mirroring is supported.
4. Compression is supported and recommended.
5. Encryption is supported.
6. Restore from URL with S3-compatible object storage has no size limitation.
7. URLs can be specified either in virtual host or path style format.
8. `WITH CREDENTIAL` is supported.
9. `REGION` is supported and the default value is `us-east-1` .
10. `MAXTRANSFERSIZE` will range from 5 MB to 20 MB. 10 MB is the default value for the S3 connector.

### Supported Arguments for Backup

| WITH options | S3 Endpoint | Notes |
| --- | --- | --- |
| BLOCKSIZE | Y | `MAXTRANSFERSIZE` will determine the Part size |
| BUFFERCOUNT | Y | |
| COMPRESSION | Y | |
| COPY\_ONLY | Y |  |
| CREDENTIAL | Y |  |
| DESCRIPTION | Y |  |
| DIFFERENTIAL | Y |  |
| ENCRYPTION | Y |  |
| FILE\_SNAPSHOT | N | |
| MAXTRANSFERSIZE | Y | From 5 MB (5,242,880 Bytes) to 20 MB (20,971,520 Bytes), default value is 10 MB (10,485,760 Bytes)|
| MEDIADESCRIPTION | Y |  |
| MEDIANAME | Y |  |
| MIRROR TO | Y |  |
| NAME | Y |  |
| NOFORMAT/FORMAT |  Y |  |
| NOINIT/INIT | N | Appending is not supported. To overwrite a backup use `WITH FORMAT`. |
| NO\_CHECKSUM/CHECKSUM | Y |  |
| NO\_TRUNCATE | Y |  |
| REGION | Y | Default value is *'us-east-1'*, must be used with *`BACKUP_OPTIONS`*|
| STATS | Y |  |

### Supported Arguments for Restore

| WITH options | S3 Endpoint | Notes |
| --- | --- | --- |
| BLOCKSIZE | Y | `MAXTRANSFERSIZE` will determine the Part size |
| BUFFERCOUNT | **N** |   |
| CHECKSUM \| NO\_CHECKSUM | Y |   |
| CREDENTIAL | Y |  |
| ENABLE\_BROKER \| ERROR\_BROKER\_CONVERSATIONS \| NEW\_BROKER | Y |  |
| FILE | **N** | Logical names not supported with `RESTORE FROM URL` |
| FILESTREAM | Y |  |
| KEEP\_CDC | Y |  |
| KEEP\_REPLICATION | Y |  |
| LOADHISTORY | Y |  |
| MAXTRANSFERSIZE | **Y** |   |
| MEDIANAME | Y |  |
| MEDIAPASSWORD | **N** | Required for some backups taken prior to SQL Server 2012 |
| MOVE | Y |  |
| PARTIAL | Y |  |
| PASSWORD | **N** | Required for some backups taken prior to SQL Server 2012 |
| RECOVERY \| NORECOVERY \| STANDBY | Y |  |
| REGION | Y | Default value is *'us-east-1'*, must be used with *`RESTORE_OPTIONS`*|
| REPLACE | Y |  |
| RESTART | Y |  |
| RESTRICTED\_USER | Y |  |
| REWIND \| NOREWIND | **N** |  |
| STATS | Y |  |
| STOP\_ON\_ERROR \| CONTINUE\_AFTER\_ERROR | Y |  |
| STOPAT \| STOPATMARK \| STOPBEFOREMARK | Y |  |
| UNLOAD \| NOUNLOAD | **N** |  |

<!-- | DBREADSIZE | **N** | Y | DBREADSIZE is not available |-->
<!-- | WITH options | Blob Storage | S3 Connector | Notes |
| --- | --- | --- | --- |
| BLOCKSIZE | Y | Y |  |
| BUFFERCOUNT | Y | Y |  |
| COMPRESSION | Y | Y | |
| COPY\_ONLY | Y | Y |  |
| CREDENTIAL | Y | Y |  |
| DESCRIPTION | Y | Y |  |
| DIFFERENTIAL | Y | Y |  |
| ENCRYPTION | Y | Y |  |
| FILE\_SNAPSHOT | Y | **N** | FILE\_SNAPSHOT is only available in Azure |
| MAXTRANSFERSIZE | Y | Y | From 5 MB (5,242,880 Bytes) to 20 MB (20,971,520 Bytes), default value is 10 MB (10,485,760 Bytes)|
| MEDIADESCRIPTION | Y | Y |  |
| MEDIANAME | Y | Y |  |
| NAME | Y | Y |  |
| NOFORMAT/FORMAT | Y | Y |  |
| NORECOVERY/STANDBY | Y | Y |  |
| NO\_CHECKSUM/CHECKSUM | Y | Y |  |
| NO\_TRUNCATE | Y | Y |  |
| REGION | N | Y | |
| STATS | Y | Y |  |
| DBREADSIZE | **N** | Y | DBREADSIZE is not available |-->

<!--An example of virtual host style format is below:

```sql
's3://<bucketName>.<virtualHost>/<pathToBackup>/<backupFileName>'
```

An example of path style format is below:

```sql
's3://<domainName>/<bucketName>/<pathToBackup>/<backupFileName>'
```

For more information see [SQL Server back up to URL for S3-compatible storage best practices and troubleshooting](sql-server-backup-to-url-s3-best-practices-and-troubleshooting.md).
|-->

### Linux support

SQL Server uses `WinHttp` to implement client of HTTP REST APIs it uses. It relies on OS certificate store for validations of the TLS certificates being presented by HTTP(s) endpoint. However, SQL Server on Linux the CA must be placed on a predefined location to be created at `/usr/local/share/ca-certificates/mssql-ca-certificates`, only the first 50 certificates can be stored and supported in this folder.

SQL Server will read the certificates from the folder during startup and add them to the trust store.

Only super user should be able to write in the folder, while the `mssql` user must be able to read.

## Unsupported features

- Backup to S3-compatible object storage with a non-secure HTTP URL is not supported. Customers are responsible for setting up their S3 host with an HTTPS URL and this endpoint will be validated by a certificate installed on the SQL Server OS host.
- Backup to S3-compatible object storage is not supported in SQL Server Express and SQL Server Express with Advanced Services editions.

<!-- ## Notebooks

* [Introducing BACKUP TO URL](nb-backup-to-url.ipynb)
* [Introducing RESTORE FROM URL](nb-restore-from-url.ipynb)
 -->

## Limitations

The following are the current limitations of backup and restore with S3-compatible object storage:

1. Due to the current limitation of S3 Standard REST API, the temporary uncommitted data files that are created in the customer's S3-compliant object store (due to an ongoing multipart upload operation) while the BACKUP T-SQL command is running, are not removed in case of failures. These uncommitted data blocks will continue to persist in the S3-compliant object storage in the case the BACKUP T-SQL command fails or is canceled. If the backup succeeds, these temporary files are removed automatically by the object store to form the final backup file. Some S3-providers will handle this through their garbage collector system.
2. The total URL length is limited to 259 characters. The full string is counted in this limitation, including the `s3://` connector name. Consequently, the usable limit is 254 characters. However, we recommend sticking to a limit of 200 characters to allow for possible introduction of query parameters.
3. The SQL credential name is limited by 128 characters in UTF-16 format.
4. Secret key ID only supports alphanumeric values.

## Examples

### Create credential

- The name of the credential must include the bucket name.
- The IDENTITY should always be `'S3 Access Key'` when using the S3 connector.
- The Access Key ID and Secret Key ID must not contain a colon.
- Only alphanumeric values are allowed.

The following examples create SQL Server credentials for authentication with the object storage endpoint:

```sql
CREATE CREDENTIAL   [s3://<endpoint>:<port>/<bucket>]
WITH    
        IDENTITY    = 'S3 Access Key',
        SECRET      = '<AccessKeyID>:<SecretKeyID>';
```

### Backup to URL

The following example performs a full database backup to the object storage endpoint, striped across multiple files:

```sql
BACKUP DATABASE <db_name>
TO      URL = 's3://<endpoint>:<port>/<bucket>/<database>_01.bak'
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_02.bak'
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_03.bak'
--
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_64.bak'
WITH    FORMAT -- overwrite
,       STATS               = 10
,       COMPRESSION;
```

### Restore from URL

The following example performs a database restore from the object storage endpoint location:

```sql
RESTORE DATABASE <db_name>
FROM    URL = 's3://<endpoint>:<port>/<bucket>/<database>_01.bak'
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_02.bak'
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_03.bak'
--
,       URL = 's3://<endpoint>:<port>/<bucket>/<database>_64.bak'
WITH    REPLACE -- overwrite
,       STATS               = 10;
```

### Options for encryption and compression

The following example shows how to backup and restore the `AdventureWorks2019` database with encryption, `MAXTRANSFERSIZE` as 20 MB and compression:

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = <password>;
GO

CREATE CERTIFICATE AdventureWorks2019Cert
    WITH SUBJECT = 'AdventureWorks2019 Backup Certificate';
GO
-- Backup database
BACKUP DATABASE AdventureWorks2019
TO URL = 's3://<endpoint>:<port>/<bucket>/AdventureWorks2019_Encrypt.bak',
WITH FORMAT, MAXTRANSFERSIZE = 20971520, COMPRESSION,
ENCRYPTION (ALGORITHM = AES_256, SERVER CERTIFICATE = AdventureWorks2019Cert)
GO

-- Restore database
RESTORE DATABASE AdventureWorks2019
FROM URL = 's3://<endpoint>:<port>/<bucket>/AdventureWorks2019_Encrypt.bak',
WITH REPLACE
```

### Using region for backup and restore

The following example shows how to backup and restore the AdventureWorks2019 database using `REGION_OPTIONS`:

```sql
-- Backup Database
BACKUP DATABASE AdventureWorks2019
TO URL = 's3://<endpoint>:<port>/<bucket>/AdventureWorks2019.bak'
WITH BACKUP_OPTIONS = '{"s3": {"region":"us-east-1"}}' 

-- Restore Database
RESTORE DATABASE AdventureWorks2019
FROM URL = 's3://<endpoint>:<port>/<bucket>/AdventureWorks2019.bak'
WITH MOVE 'AdventureWorks2019' TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\AdventureWorks2019.mdf'
, MOVE 'AdventureWorks2019_log' TO 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA\AdventureWorks2019.ldf'
, RESTORE_OPTIONS = '{"s3": {"region":"us-east-1"}}' 
```

## Next steps

- [SQL Server back up to URL for S3-compatible object storage best practices and troubleshooting](sql-server-backup-to-url-s3-compatible-object-storage-best-practices-and-troubleshooting.md)
- [SQL Server back up to URL for Microsoft Azure Blob Storage best practices and troubleshooting](sql-server-backup-to-url-best-practices-and-troubleshooting.md)

<!-- -[Tutorial: SQL Backup and restore with object storage](sql-server-backup-to-url-s3-compatible-storage.md)-->

<!-- Internal LInks -->

<!-- Public Links -->
[docs_backup_best]: (https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)
[MAXTRANSFERSIZE]: (https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql?view=sql-server-ver15#with-options)

[Azure block blobs]: (https://docs.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)
