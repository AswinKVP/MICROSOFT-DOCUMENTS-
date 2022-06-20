# Virtualize CSV file

SQL Server 2022 can virtualize data from CSV files. This process allows the data to stay in its original location, but can be queried from a SQL Server instance with T-SQL commands, like any other table. This feature uses PolyBase connectors, and minimizes the need for ETL processes.

In the example below, the csv file is stored on Azure Blob Storage.

For more information on data virtualization, [Introducing data virtualization with PolyBase](polybase-guide.md).

## Pre-configuration

### 1. Enable PolyBase in `sp_configure`

```sql
exec sp_configure @configname = 'polybase enabled', @configvalue = 1;

RECONFIGURE;

```

### 2. Create a user database

User database be used to work with the data and store the scoped credential. In this example *[CSV_Demo]* will be used.

```sql
CREATE DATABASE [CSV_Demo];
```

### 3. Create a master key

The master key is required to encrypt the credential secret:

```sql
USE [CSV_Demo];

CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'S0me!nfo';  

```

### 4. Create Database Scoped Credential

Database scoped credential will be used for the External Data Source. In this example the CSV file stays on Azure Blob Storage, prefix *'abs'*, and  *'SHARED ACCESS SIGNATURE'* is the identity method.

For more information about the connectors and its prefixes check [Create External Data Source](create-external-data-source-sql.md).

```sql
CREATE EXTERNAL DATA SOURCE Blob_CSV
WITH
(
 LOCATION = 'abs://<container>@<storage_account>.blob.core.windows.net'
,CREDENTIAL = blob_storage
)
```

### 5. Use OPENROWQUERY to access the data

In this example the file is named *'call_center.csv'* and the data starts on the second row.

```sql
SELECT  * 
FROM    OPENROWSET
        (   BULK 'call_center.csv'
        ,   FORMAT = 'CSV'
        ,   DATA_SOURCE = 'Blob_CSV'
        ,   FIRSTROW    = 2
        )
WITH    (   cc_call_center_sk         integer                       ,
            cc_call_center_id         char(16)                      ,
            cc_rec_start_date         date                          ,
            cc_rec_end_date           date                          ,
            cc_closed_date_sk         integer                       ,
            cc_open_date_sk           integer                       ,
            cc_name                   varchar(50)                   ,
            cc_class                  varchar(50)                   ,
            cc_employees              integer                       ,
            cc_sq_ft                  integer                       ,
            cc_hours                  char(20)                      ,
            cc_manager                varchar(40)                   ,
            cc_mkt_id                 integer                       ,
            cc_mkt_class              char(50)                      ,
            cc_mkt_desc               varchar(100)                  ,
            cc_market_manager         varchar(40)                   ,
            cc_division               integer                       ,
            cc_division_name          varchar(50)                   ,
            cc_company                integer                       ,
            cc_company_name           char(50)                      ,
            cc_street_number          char(10)                      ,
            cc_street_name            varchar(60)                   ,
            cc_street_type            char(15)                      ,
            cc_suite_number           char(10)                      ,
            cc_city                   varchar(60)                   ,
            cc_county                 varchar(30)                   ,
            cc_state                  char(2)                       ,
            cc_zip                    char(10)                      ,
            cc_country                varchar(20)                   ,
            cc_gmt_offset             decimal(5,2)                  ,
            cc_tax_percentage         decimal(5,2)                  
        )
        AS [cc];
```

### 6. External Table

Create External Table can also be used to virtualize the csv data in SQL Server.

#### Important notes

1. The columns have to be defined and strongly typed
2. Nullability and collation can be defined at the column level
3. Statistics can be created and stored on the external table

#### Benefits

While external tables take more effort to create, they also provide additional benefits over OPENROWSET.

1. You can strengthen the definition of the data typing for a given column.
2. Define NULLability.
3. Define COLLATION.
4. You can create statistics for a column to optimize the quality of the query plan
5. You can create a more granular model within SQL Server for data access to enhance your security model

For the following example the same data source will be used.

### 6.1 Create External File Format

For the purpose of file formatting and first row definition External File Format is required, in the following example the data starts on the second row. External File Formats are also recommended due to reusability.

```sql

CREATE EXTERNAL FILE FORMAT csv_ff
WITH
(   FORMAT_TYPE = DELIMITEDTEXT
,   FORMAT_OPTIONS  (    FIELD_TERMINATOR = ','
                    ,    STRING_DELIMITER = '"'
                  ,    FIRST_ROW = 2
                    )
);
```

### 6.2 Create External Table

```sql
CREATE EXTERNAL TABLE extCall_Center_csv
(
            cc_call_center_sk         integer             NOT NULL  ,
            cc_call_center_id         char(16)            NOT NULL  ,
            cc_rec_start_date         date                          ,
            cc_rec_end_date           date                          ,
            cc_closed_date_sk         integer                       ,
            cc_open_date_sk           integer                       ,
            cc_name                   varchar(50)                   ,
            cc_class                  varchar(50)                   ,
            cc_employees              integer                       ,
            cc_sq_ft                  integer                       ,
            cc_hours                  char(20)                      ,
            cc_manager                varchar(40)                   ,
            cc_mkt_id                 integer                       ,
            cc_mkt_class              char(50)                      ,
            cc_mkt_desc               varchar(100)                  ,
            cc_market_manager         varchar(MAX)                  ,
            cc_division               varchar(50)                   ,
            cc_division_name          varchar(50)                   ,
            cc_company                varchar(60)                   ,
            cc_company_name           char(50)                      ,
            cc_street_number          char(10)                      ,
            cc_street_name            varchar(60)                   ,
            cc_street_type            char(15)                      ,
            cc_suite_number           char(10)                      ,
            cc_city                   varchar(60)                   ,
            cc_county                 varchar(30)                   ,
            cc_state                  char(20)                      ,
            cc_zip                    char(20)                      ,
            cc_country                varchar(MAX)                  ,
            cc_gmt_offset             decimal(5,2)                  ,
            cc_tax_percentage         decimal(5,2) 
)
WITH
(
    LOCATION = 'call_center.csv',
    DATA_SOURCE = Blob_CSV
    ,FILE_FORMAT = csv_ff
)
GO
```


## Next Steps

Learn more about related concepts in the following articles:

- [CREATE EXTERNAL DATA SOURCE](../../t-sql/statements/create-external-data-source-transact-sql.md)
- [CREATE EXTERNAL FILE FORMAT](../../t-sql/statements/create-external-file-format-transact-sql.md)
- [CREATE EXTERNAL TABLE](../../t-sql/statements/create-external-table-transact-sql.md)
  