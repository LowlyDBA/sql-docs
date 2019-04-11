---
title: "What's new in SQL Server 2019 | Microsoft Docs"
ms.date: 03/27/2018
ms.prod: "sql-server-2019"
ms.reviewer: ""
ms.technology: release-landing
ms.topic: "article"
author: MikeRayMSFT
ms.author: mikeray
manager: craigg
monikerRange: ">=sql-server-ver15||=sqlallproducts-allversions"
---
# What's new in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)]

[!INCLUDE[tsql-appliesto-ssver15-xxxx-xxxx-xxx](../includes/tsql-appliesto-ssver15-xxxx-xxxx-xxx.md)]

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] builds on previous releases to grow SQL Server as a platform that gives you choices of development languages, data types, on-premises or cloud, and operating systems. This article summarizes what is new for [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)]. The first section identifies features that are added in the latest preview release. The other sections of this article provide details about all of the features released to date for this [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)].

For more information and known issues, see the [[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Release Notes](sql-server-ver15-release-notes.md).

**Try [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)]!**

- [![Download from Evaluation Center](../includes/media/download2.png)](https://go.microsoft.com/fwlink/?LinkID=862101) [Download [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] to install on Windows](https://go.microsoft.com/fwlink/?LinkID=862101).
- Install on Linux for [Red Hat Enterprise Server](../linux/quickstart-install-connect-red-hat.md), [SUSE Linux Enterprise Server](../linux/quickstart-install-connect-suse.md), and [Ubuntu](../linux/quickstart-install-connect-ubuntu.md).
- [Run on [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] on Docker](../linux/quickstart-install-connect-docker.md).

**Use the [latest tools](#tools) for the best experience with [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)].**

## CTP 2.4

Community technology preview (CTP) 2.4 is the latest public release of [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)]. This release includes improvements from previous CTP releases to fix bugs, improve security, and optimize performance. In addition, the following features are added or enhanced for [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] CTP 2.4.

- [Big data cluster](#bigdatacluster)
  - Guidance on GPU support for running deep learning with TensorFlow in Spark.
  - Spark runtime upgrade to Spark 2.4.
  - `INSERT INTO SELECT` support for the data pool.
  - `FORCE SCALEOUTEXECUTION` and `DISABLE SCALEOUTEXECUTION` option clause for external table queries.

- [Database engine](#databaseengine)
  - Truncation error message defaults to include table and column names, and truncated value. See [Truncation](#truncation).
  - New DMF `sys.dm_exec_query_plan_stats` returns the equivalent of the last known actual execution plan for most queries.
  - Transparent Data Encryption (TDE) scan - suspend and resume.

- [SQL Server Analysis Services](#ssas)
  - Many-to-many relationships in tabular models.
  - Property settings for resource governance.

The following sections describe the new features that have been introduced in previous releases of [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md) .

## <a id="bigdatacluster"></a>Big data clusters

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] [Big data clusters](../big-data-cluster/big-data-cluster-overview.md) enables new scenarios including the following:

- [GPU support for running deep learning with TensorFlow in Spark](../big-data-cluster/spark-gpu-tensorflow.md). (CTP 2.4)
- Spark runtime upgrade to Spark 2.4. (CTP 2.4)
- `INSERT INTO SELECT` support for the data pool.
- `FORCE SCALEOUTEXECUTION` and `DISABLE SCALEOUTEXECUTION` option clause for external table queries.
- [Submit Spark jobs on [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] big data clusters in IntelliJ](../big-data-cluster/spark-submit-job-intellij-tool-plugin.md). (CTP 2.3)
- [Application deployment and management experience](../big-data-cluster/big-data-cluster-create-apps.md) for a variety of data-related apps, including operationalizing machine learning models using R and Python, running SQL Server Integration Services (SSIS) jobs, and more. (CTP 2.3)
- [Use Sparklyr in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] big data clusters](../big-data-cluster/sparklyr-from-RStudio.md). (CTP 2.3)
- Mount external HDFS-compatible storage into big data cluster with [HDFS tiering](../big-data-cluster/hdfs-tiering.md). (CTP 2.3)
- Use SparkR from Azure Data Studio on a big data cluster. (CTP 2.2)
- [Deploy Python and R apps](../big-data-cluster/big-data-cluster-create-apps.md). (CTP 2.2)
- Deploy a Big Data cluster with [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] and Spark Linux containers on Kubernetes. (CTP 2.0)
- Access your big data from HDFS. (CTP 2.0)
- Run Advanced analytics and machine learning with Spark. (CTP 2.0)
- Use Spark streaming to data to SQL data pools. (CTP 2.0)
- Run Query books that provide a notebook experience in [**Azure Data Studio**](../sql-operations-studio/what-is.md). (CTP 2.0)
 
[!INCLUDE [Big data clusters preview](../includes/big-data-cluster-preview-note.md)]

## <a id="databaseengine"></a> Database Engine

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces or enhances the following new features for the [!INCLUDE[ssDEnoversion](../includes/ssdenoversion-md.md)].

### New query_post_execution_plan_profile Extended Event (CTP 2.4)

The new `query_post_execution_plan_profile` Extended Event collects the equivalent of an actual execution plan based on lightweight profiling, unlike `query_post_execution_showplan` which uses standard profiling. For more information, see [Query profiling infrastructure](../relational-databases/performance/query-profiling-infrastructure.md).

#### Example 1 - Extended Event session using standard profiling

```sql
CREATE EVENT SESSION [QueryPlanOld] ON SERVER 
ADD EVENT sqlserver.query_post_execution_showplan(
    ACTION(sqlos.task_time, sqlserver.database_id, 
    sqlserver.database_name, sqlserver.query_hash_signed, 
    sqlserver.query_plan_hash_signed, sqlserver.sql_text))
ADD TARGET package0.event_file(SET filename = N'C:\Temp\QueryPlanStd.xel')
WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, 
    MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB, 
    MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF, STARTUP_STATE=OFF);
```

#### Example 2 - Extended Event session using lightweight profiling

```sql
CREATE EVENT SESSION [QueryPlanLWP] ON SERVER 
ADD EVENT sqlserver.query_post_execution_plan_profile(
    ACTION(sqlos.task_time, sqlserver.database_id, 
    sqlserver.database_name, sqlserver.query_hash_signed, 
    sqlserver.query_plan_hash_signed, sqlserver.sql_text))
ADD TARGET package0.event_file(SET filename=N'C:\Temp\QueryPlanLWP.xel')
WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, 
    MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB, 
    MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF, STARTUP_STATE=OFF);
```

### New DMF sys.dm_exec_query_plan_stats (CTP 2.4) 

The new DMF `sys.dm_exec_query_plan_stats` returns the equivalent of the last known actual execution plan for most queries, based on lightweight profiling. For more information, see [sys.dm_exec_query_plan_stats](../relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql.md) and [Query profiling infrastructure](../relational-databases/performance/query-profiling-infrastructure.md). See the following script as an example:

```sql
SELECT *
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_query_plan_stats(plan_handle)
WHERE objtype ='Trigger';
GO
```

This is an opt-in feature and requires [trace flag](../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md) 2451 to be enabled.

### Transparent Data Encryption (TDE) scan - suspend and resume (CTP 2.4)

In order to enable [Transparent Data Encryption (TDE)](../relational-databases/security/encryption/transparent-data-encryption.md) on a database, [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] must perform an encryption scan which reads each page from the data file(s) into the buffer pool, and then writes the encrypted pages back out to disk. To provide the user with more control over the encryption scan, [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces TDE scan - suspend and resume syntax so that you can pause the scan while the workload on the system is heavy, or during business-critical hours, and then resume the scan later.

Use the following syntax to pause the TDE encryption scan:

```sql
ALTER DATABASE <db_name> SET ENCRYPTION SUSPEND;
```

Similarly, the following syntax resumes the TDE encryption scan:

```sql
ALTER DATABASE <db_name> SET ENCRYPTION RESUME;
```

To show the current state of the encryption scan, `encryption_scan_state` has been added to the `sys.dm_database_encryption_keys` dynamic management view. There is also a new column called `encryption_scan_modify_date` which will contain the date and time of the last encryption scan state change. Also note that if the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instance is restarted while the encryption scan is in a suspended state, a message will be logged in the errorlog on startup indicating that there is an existing scan which has been paused.

### Accelerated database recovery (CTP 2.3)

[Accelerated database recovery](/azure/sql-database/sql-database-accelerated-database-recovery/) greatly improves database availability, especially in the presence of long running transactions, by redesigning the SQL Server database engine recovery process. [Database recovery](../relational-databases/logs/the-transaction-log-sql-server.md?#recovery-of-all-incomplete-transactions-when--is-started) is the process SQL Server uses for each database to start at a transactionally consistent - or clean - state. A database, with accelerated database recovery enabled, completes recovery significantly faster after a fail over or other non-clean shutdown. As of CTP 2.3, accelerated database recovery can be enabled per-database using the following syntax:

```sql
ALTER DATABASE <db_name> SET ACCELERATED_DATABASE_RECOVERY = {ON | OFF}
```

> [!NOTE]
> This syntax is not required to take advantage of this feature in Azure SQL DB, where it is on by default.

If you have critical databases that are prone to large transactions, experiment with this feature during the preview. Provide feedback to [[!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] team](<https://aka.ms/sqlfeedback>).

### Query Store plan forcing support for fast forward and static cursors (CTP 2.3)

Query Store now supports the ability to force query execution plans for fast forward and static T-SQL and API cursors. Forcing is now supported via `sp_query_store_force_plan` or through SQL Server Management Studio Query Store reports.

### Reduced recompilations for workloads using temporary tables across multiple scopes (CTP 2.3)

Prior to this feature, when referencing a temporary table with a data manipulation language (DML) statement (`SELECT`, `INSERT`, `UPDATE`, `DELETE`), if the temporary table was created by an outer scope batch, this would result in a recompile of the DML statement each time it is executed. With this improvement, SQL Server performs additional lightweight checks to avoid unnecessary recompilations:

- Check if the outer-scope module used for creating the temporary table at compile time is the same one used for consecutive executions. 
- Keep track of any data definition language (DDL) changes made at initial compilation and  compare them with DDL operations for consecutive executions. 

The end result is a reduction in extraneous recompilations and CPU-overhead.

### Improved indirect checkpoint scalability (CTP 2.3)

In previous versions of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], users may experience non-yielding scheduler errors when there is a database that generates a large number of dirty pages, such as tempdb. [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces improved scalability for Indirect Checkpoint, which should help avoid these errors on databases that have a heavy UPDATE/INSERT workload.

### UTF-8 support (CTP 2.3)

Full support for the widely used UTF-8 character encoding as an import or export encoding, or as database-level or column-level collation for text data. UTF-8 is allowed in the `CHAR` and `VARCHAR` datatypes, and is enabled when creating or changing an object's collation to a collation with the `UTF8` suffix. 

For example,`LATIN1_GENERAL_100_CI_AS_SC` to `LATIN1_GENERAL_100_CI_AS_SC_UTF8`. UTF-8 is only available to Windows collations that support supplementary characters, as introduced in [!INCLUDE[ssSQL11](../includes/sssql11-md.md)]. `NCHAR` and `NVARCHAR` allow UTF-16 encoding only, and remain unchanged.

This feature may provide significant storage savings, depending on the character set in use. For example, changing an existing column data type with ASCII (Latin) strings from `NCHAR(10)` to `CHAR(10)` using an UTF-8 enabled collation, translates into 50% reduction in storage requirements. This reduction is because `NCHAR(10)` requires 20 bytes for storage, whereas `CHAR(10)` requires 10 bytes for the same Unicode string.

For more information, see [Collation and Unicode Support](../relational-databases/collations/collation-and-unicode-support.md).

**CTP 2.1** Adds support to select UTF-8 collation as default during [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] setup.

**CTP 2.2** Adds support to use UTF-8 character encoding with SQL Server Replication.

**CTP 2.3** Adds support to use UTF-8 character encoding with a BIN2 collation (UTF8_BIN2).

### Scalar UDF inlining (CTP 2.1)

Scalar UDF inlining automatically transforms scalar user-defined functions (UDF) into relational expressions and embeds them in the calling SQL query, thereby improving the performance of workloads that leverage scalar UDFs. Scalar UDF inlining facilitates cost-based optimization of operations inside UDFs, and results in efficient plans that are set-oriented and parallel as opposed to inefficient, iterative, serial execution plans. This feature is enabled by default under database compatibility level 150.

For more information, see [Scalar UDF inlining](../relational-databases/user-defined-functions/scalar-udf-inlining.md).

### <a name="truncation" />Truncation error message improved to include table and column names, and truncated value (CTP 2.1)

The error message ID 8152 `String or binary data would be truncated` is familiar to many [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] developers and administrators who develop or maintain data movement workloads; the error is raised during data transfers between a source and a destination with different schemas when the source data is too large to fit into the destination data type. This error message can be time-consuming to troubleshoot. [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces a new, more specific error message (2628) for this scenario:  

`String or binary data would be truncated in table '%.*ls', column '%.*ls'. Truncated value: '%.*ls'.`

The new error message 2628 provides more context for the data truncation problem, simplifying the troubleshooting process. 

**CTP 2.1 and CTP 2.2** This is an opt-in error message and requires [trace flag](../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md) 460 to be enabled.

**CTP 2.4** Error message 2628 becomes the default truncation message and replaces error message 8152 under database compatibility level 150. A new database scoped configuration `VERBOSE_TRUNCATION_WARNINGS` is introduced to switch between error message 2628 and 8152 when the database compatibility level is 150. For more information, see [ALTER DATABASE SCOPED CONFIGURATION](../t-sql/statements/alter-database-scoped-configuration-transact-sql.md).
For database compatibility level 140 or lower, error message 2628 remains an opt-in error message that requires [trace flag](../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md) 460 to be enabled.

### Improved diagnostic data for stats blocking (CTP 2.1)

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] provides improved diagnostic data for long-running queries that wait on synchronous statistics update operations. The dynamic management view `sys.dm_exec_requests` column `command` shows `SELECT (STATMAN)` if a `SELECT` is waiting for a synchronous statistics update operation to complete prior to continuing query execution. Additionally, the new wait type `WAIT_ON_SYNC_STATISTICS_REFRESH` is surfaced in the `sys.dm_os_wait_stats` dynamic management view. It shows the accumulated instance-level time spent on synchronous statistics refresh operations.

### Hybrid buffer pool (CTP 2.1)

Hybrid buffer pool is a new feature of the SQL Server database engine where database pages sitting on database files placed on a persistent memory (PMEM) device will be directly accessed when required. Since PMEM devices provide very low latency for data access, the engine can forgo making a copy of the data in a "clean pages" area of the buffer pool and simply access the page directly on PMEM. Access is performed using memory mapped I/O, as is the case with enlightenment. This brings performance benefits from avoiding a copy of the page to DRAM, and from the avoidance of the I/O stack of the operating system to access the page on persistent storage. This feature is available on both SQL Server on Windows and SQL Server on Linux.

For more information, see [Hybrid buffer pool](../database-engine/configure-windows/hybrid-buffer-pool.md)

### Static data masking (CTP 2.1)

[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces static data masking. You can use static data masking to sanitize sensitive data in copies of SQL Server databases. Static data masking helps create a sanitized copy of databases where all sensitive information has been altered in a way that makes the copy sharable with non-production users. Static data masking can be used for development, testing, analytics and business reporting, compliance, troubleshooting, and any other scenario where specific data cannot be copied to different environments.

Static data masking operates at the column level. Select which columns to mask, and for each column selected, specify a masking function. Static data masking copies the database and then applies the specified masking functions to the columns.

#### Static data masking vs. dynamic data masking

Data masking is the process of applying a mask on a database to hide sensitive information and replacing it with new data or scrubbed data. Microsoft offers two masking options, static data masking and dynamic data masking. Dynamic data masking was introduced in [!INCLUDE[ssSQL16](../includes/sssql16-md.md)]. The following table compares these two solutions:

|Static data masking |Dynamic data masking|
|:----|:----|
|Happens on a copy of the database <br/><br/>Original data not retrievable<br/><br/>Mask occurs at the storage level<br/><br/>All users have access to the same masked data<br/><br/>Geared toward continuous team-wide access|Happens on the original database<br/><br/>Original data intact<br/><br/>Mask occurs on-the-fly at query time<br/><br/>Mask varies based on user permission <br/><br/>Geared toward punctual user-specific access|

### Database compatibility level (CTP 2.0)

Database **COMPATIBILITY_LEVEL 150** is added. To enable for a specific user database, execute:

   ```sql
   ALTER DATABASE database_name SET COMPATIBILITY_LEVEL =  150;
   ```

### Resumable online index create (CTP 2.0)

**Resumable online index create** allows an index create operation to pause and resume later from where the operation was paused or failed, instead of restarting from the beginning.

Resumable online index create supports the follow scenarios:
- Resume an index create operation after an index create failure, such as after a database failover or after running out of disk space.
- Pause an ongoing index create operation and resume it later allowing to temporarily free system resources as required and resume this operation later.
- Create large indexes without using as much log space and a long-running transaction that blocks other maintenance activities and allowing log truncation.

In case of an index create failure, without this feature an online index create operation must be executed again and the operation must be restarted from the beginning.

With this release, we extend the resumable functionality adding this feature to available [resumable online index rebuild](http://azure.microsoft.com/blog/modernize-index-maintenance-with-resumable-online-index-rebuild/).

In addition, this feature can be set as the default for a specific database using [database scoped default setting for online and resumable DDL operations](../t-sql/statements/alter-database-scoped-configuration-transact-sql.md).

For more information, see [Resumable Online Index Create](../t-sql/statements/create-index-transact-sql.md#resumable-indexes).

### Build and rebuild clustered columnstore indexes online (CTP 2.0)

Convert row-store tables into columnstore format. Creating clustered columnstore indexes (CCI) was an offline process in the previous versions of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] - requiring all changes stop while the CCI is created. With [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] and 
[!INCLUDE[ssSDSfull](../includes/sssdsfull-md.md)] you can create or re-create CCI online. Workload will not be blocked and all changes made on the underlying data are transparently added into the target columnstore table. Examples of new [!INCLUDE[tsql](../includes/tsql-md.md)] statements that can be used are:

  ```sql
  CREATE CLUSTERED COLUMNSTORE INDEX cci
    ON <tableName>
    WITH (ONLINE = ON);
  ```

  ```sql
  ALTER INDEX cci
    ON <tableName>
    REBUILD WITH (ONLINE = ON);
  ```

### Always Encrypted with secure enclaves (CTP 2.0)

Expands upon Always Encrypted with in-place encryption and rich computations. The expansions come from the enabling of computations on plaintext data, inside a secure enclave on the server side.

Cryptographic operations include the encryption of columns, and the rotating of column encryption keys. These operations can now be issued by using [!INCLUDE[tsql](../includes/tsql-md.md)], and they do not require that data be moved out of the database. Secure enclaves provide Always Encrypted to a broader set of scenarios that have both of the following requirements:  

- The demand that sensitive data are protected from high-privilege, yet unauthorized users, including database administrators, system administrators, cloud operators, or malware.
- The requirement that rich computations on protected data be supported within the database system.

For details, see [Always Encrypted with secure enclaves](../relational-databases/security/encryption/always-encrypted-enclaves.md).

> [!NOTE]
> Always Encrypted with secure enclaves is only available on Windows OS.

### Intelligent query processing (CTP 2.0)

- **Row mode memory grant feedback** expands on the memory grant feedback feature introduced in [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] by adjusting memory grant sizes for both batch and row mode operators. For an excessive memory grant condition, if the granted memory is more than two times the size of the actual used memory, memory grant feedback will recalculate the memory grant. Consecutive executions will then request less memory. For an insufficiently sized memory grant that results in a spill to disk, memory grant feedback will trigger a recalculation of the memory grant. Consecutive executions will then request more memory. This feature is enabled by default under database compatibility level 150.

- **Approximate COUNT DISTINCT** returns the approximate number of unique non-null values in a group. This function is designed for use in big data scenarios. This function is optimized for queries where all the following conditions are true:
   - Accesses data sets of at least millions of rows.
   - Aggregates a column or columns that have a large number of distinct values.
   - Responsiveness is more critical than absolute precision.
      - `APPROX_COUNT_DISTINCT` returns results that are typically within 2% of the precise answer.
      - And it returns the approximate answer in a small fraction of the time needed for the precise answer.

- **Batch mode on rowstore** no longer requires a columnstore index to process a query in batch mode. Batch mode allows query operators to work on a set of rows, instead of just one row at a time. This feature is enabled by default under database compatibility level 150. Batch mode improves the speed of queries that access rowstore tables when all the following are true:
   - The query uses analytic operators such as joins or aggregation operators.
   - The query involves 100,000 or more rows.
   - The query is CPU bound, rather than input/output data bound.
   - Creation and use of a columnstore index would have one of the following drawbacks:
      - Would add too much overhead to the query.
      - Or, is not feasible because your application depends on a feature that is not yet supported with columnstore indexes.

- **Table variable deferred compilation** improves plan quality and overall performance for queries referencing table variables. During optimization and initial compilation, this feature will propagate cardinality estimates that are based on actual table variable row counts. This accurate row count information will be used for optimizing downstream plan operations. This feature is enabled by default under database compatibility level 150.

To use intelligent query processing features, set database `COMPATIBILITY_LEVEL = 150`.

### <a id="programmability"></a> Java language programmability extensions (CTP 2.0)

- **Java language extension (preview)**: Use the Java language extension to execute Java code in [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)]. In [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)], this extension is installed when you add the feature 'Machine Learning Services (in-database)' to your [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instance.

### <a id="sqlgraph"></a> SQL Graph features (CTP 2.3)

- **Use derived table or view aliases in graph match query (CTP 2.1)** Graph queries on [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] preview support using view and derived table aliases in the `MATCH` syntax. To use these aliases in `MATCH`, the views and derived tables must be created on either a set of node or a set of edge tables, using the `UNION ALL` operator. The node or edge tables may or may not have filters on it. The ability to use derived table and view aliases in `MATCH` queries can be very useful in scenarios where you are looking to query heterogeneous entities or heterogeneous connections between two or more entities in your graph.

- **Match support in `MERGE` DML (CTP 2.0)** allows you to specify graph relationships in a single statement, instead of separate `INSERT`, `UPDATE`, or `DELETE` statements. Merge your current graph data from node or edge tables with new data using the `MATCH` predicates in the `MERGE` statement. This feature enables `UPSERT` scenarios on edge tables. Users can now use a single merge statement to insert a new edge or update an existing one between two nodes.

- **Edge constraints (CTP 2.0)** are introduced for edge tables in SQL Graph. Edge tables can connect any node to any other node in the database. With introduction of edge constraints, you can now apply some restrictions on this behavior. The new `CONNECTION` constraint can be used to specify the type of nodes a given edge table will be allowed to connect to in the schema. 

  **(CTP 2.3)** Extending this feature further, you can define cascaded delete actions on an edge constraint. You can define the actions that the database engine takes when a user deletes the node(s), that a given edge connects.

### Database scoped default setting for online and resumable DDL operations (CTP 2.0)

- **Database scoped default setting for online and resumable DDL operations** allows a default behavior setting for `ONLINE` and `RESUMABLE` index operations at the database level, rather than defining these options for each individual index DDL statement such as index create or rebuild.

- Set these defaults using the `ELEVATE_ONLINE` and `ELEVATE_RESUMABLE` database scoped configuration options. Both options will cause the engine to automatically elevate supported operations to index online or resumable execution. You can enable the following behaviors using these options:

  - `FAIL_UNSUPPORTED` option allows all index operations online or resumable and fail index operations that are not supported for online or resumable.
  - `WHEN_SUPPPORTED` option allows supported operations online or resumable and run index unsupported operations offline or non-resumable.
  - `OFF` option allows the current behavior of executing all index operations offline and non-resumable unless explicitly specified in the DDL statement.

To override the default setting, include the `ONLINE` or `RESUMABLE` option in the index create and rebuild commands. 

Without this feature, you have to specify the online and resumable options directly in the index DDL statement such as index create and rebuild.

For more information on index resumable operations, see [Resumable Online Index Create](https://azure.microsoft.com/blog/resumable-online-index-create-is-in-public-preview-for-azure-sql-db/).

### <a id="ha"></a>Always On Availability Groups - more synchronous replicas (CTP 2.0)

- **Up to five synchronous replicas**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] increases the maximum number of synchronous replicas to 5, up from 3 in [!INCLUDE[ssSQL17](../includes/sssql17-md.md)]. You can configure this group of five replicas to have automatic failover within the group. There is one primary replica, plus four synchronous secondary replicas.

- **Secondary-to-primary replica connection redirection**: Allows client application connections to be directed to the primary replica regardless of the target server specified in the connection string. This capability allows connection redirection without a listener. Use secondary-to-primary replica connection redirection in the following cases:

  - The cluster technology does not offer a listener capability.
  - A multi subnet configuration where redirection becomes complex.
  - Read scale-out or disaster recovery scenarios where cluster type is `NONE`.

For details, see [Secondary to primary replica read/write connection redirection (Always On Availability Groups)](../database-engine/availability-groups/windows/secondary-replica-connection-redirection-always-on-availability-groups.md).

### Data discovery and classification (CTP 2.0)

Data discovery and classification provides advanced capabilities that are natively built into [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)]. Classifying and labeling your most sensitive data provides the following benefits:
- Helps meet data privacy standards and regulatory compliance requirements.
- Supports security scenarios, such as monitoring (auditing), and alerting on anomalous access to sensitive data.
- Makes it easier to identify where sensitive data resides in the enterprise, so that administrators can take the right steps to secure the database.

For more information, see [SQL Data Discovery and Classification](../relational-databases/security/sql-data-discovery-and-classification.md).

[Auditing](../relational-databases/security/auditing/sql-server-audit-database-engine.md) has also been enhanced to include a new field in the audit log called `data_sensitivity_information`, which logs the sensitivity classifications (labels) of the actual data that was returned by the query. For details and examples, see [Add sensitivity classification](../t-sql/statements/add-sensitivity-classification-transact-sql.md).

>[!NOTE]
>There are no changes in terms of how audit is enabled. There is a new field added to the audit records, `data_sensitivity_information`, which logs the sensitivity classifications (labels) of the actual data that was returned by the query. See [Auditing access to sensitive data](/azure/sql-database/sql-database-data-discovery-and-classification/#subheading-3).

### Expanded support for persistent memory devices (CTP 2.0)

Any [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] file that is placed on a persistent memory device can now operate in *enlightened* mode. [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] directly accesses the device, bypassing the storage stack of the operating system using efficient memcpy operations. This mode improves performance because it allows low latency input/output against such devices.
    - Examples of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] files include:
        - Database files
        - Transaction log files
        - In-Memory OLTP checkpoint files
    - Persistent memory is also known as storage class memory.
    - Persistent memory is occasionally referred to informally as *pmem* on some non-Microsoft websites.

> [!NOTE]
> For this preview release, enlightenment of files on persistent memory devices is only available on Linux. [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] on Windows supports persistent memory devices starting with [!INCLUDE[ssSQL15](../includes/sssql15-md.md)].

### Support for columnstore statistics in DBCC CLONEDATABASE (CTP 2.0)

`DBCC CLONEDATABASE` creates a schema-only copy of a database that includes all the elements necessary to troubleshoot query performance issues without copying the data. In previous versions of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], the command did not copy the statistics necessary to accurately troubleshoot columnstore index queries and manual steps were required to capture this information. Now in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)], `DBCC CLONEDATABASE` automatically captures the stats blobs for columnstore indexes, so no manual steps will be required.

### New options added to sp_estimate_data_compression_savings (CTP 2.0)

`sp_estimate_data_compression_savings` returns the current size of the requested object and estimates the object size for the requested compression state. Currently this procedure supports three options: `NONE`, `ROW`, and `PAGE`. [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces two new options: `COLUMNSTORE` and `COLUMNSTORE_ARCHIVE`. These new options will allow you to estimate the space savings if a columnstore index is created on the table using either standard or archive columnstore compression.

### <a id="ml"></a> SQL Server Machine Learning Services failover clusters and partition based modeling (CTP 2.0)

- **Partition-based modeling**: Process external scripts per partition of your data using the new parameters added to `sp_execute_external_script`. This functionality supports training many small models (one model per partition of data) instead of one large model.

- **Windows Server Failover Cluster**: Configure high availability for Machine Learning Services on a Windows Server Failover Cluster.

For detailed information, see [What's new in SQL Server Machine Learning Services](../advanced-analytics/what-s-new-in-sql-server-machine-learning-services.md).

### Lightweight query profiling infrastructure enabled by default (CTP 2.0)

The lightweight query profiling infrastructure (LWP) provides query performance data more efficiently than standard profiling mechanisms. Lightweight profiling is now enabled by default. It was introduced in [!INCLUDE[ssSQL15](../includes/sssql15-md.md)] SP1. Lightweight profiling offers a query execution statistics collection mechanism with an expected overhead of 2% CPU, compared with an overhead of up to 75% CPU for the standard query profiling mechanism. On previous versions, it was OFF by default. Database administrators could enable it with [trace flag 7412](../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md). 

For more information on lightweight profiling, see [Query Profiling Infrastructure](../relational-databases/performance/query-profiling-infrastructure.md).

**CTP 2.3** A new database scoped configuration `LIGHTWEIGHT_QUERY_PROFILING` is introduced to enable or disable the lightweight query profiling infrastructure.

### <a id="polybase"></a>New PolyBase connectors

- **New connectors for [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], Oracle, Teradata, and MongoDB**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] introduces new connectors to external data for [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)], Oracle, Teradata, and MongoDB.

### New sys.dm_db_page_info system function returns page information (CTP 2.0)

`sys.dm_db_page_info(database_id, file_id, page_id, mode)` returns information about a page in a database. The function returns a row that contains the header information from the page, including the `object_id`, `index_id`, and `partition_id`. This function replaces the need to use `DBCC PAGE` in most cases. 

In order to facilitate troubleshooting of page-related waits, a new column called page_resource was also added to `sys.dm_exec_requests` and `sys.sysprocesses`. This new column allows you to join `sys.dm_db_page_info` to these views via another new system function - `sys.fn_PageResCracker`. See the following script as an example:

```sql
SELECT page_info.* 
FROM sys.dm_exec_requests AS d 
  CROSS APPLY sys.fn_PageResCracker(d.page_resource) AS r
  CROSS APPLY sys.dm_db_page_info(r.db_id, r.file_id, r.page_id,'DETAILED')
    AS page_info;
```

## <a id="sqllinux"></a> SQL Server on Linux

- **Always On Availability Group on Docker containers with Kubernetes (CTP 2.2)**: Kubernetes can orchestrate containers running [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instances to provide a highly available set of databases with SQL Server Always On Availability Groups. A Kubernetes operator deploys a StatefulSet including a container with **mssql-server container**, and a health monitor.

- **New container registry (CTP 2.1)**: All container images for [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] as well as [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] are now located in the Microsoft Container Registry. Microsoft Container Registry is the official container registry for the distribution of Microsoft product containers. In addition, certified RHEL-based images are now published.

  - Microsoft Container Registry: `mcr.microsoft.com/mssql/server:vNext-CTP2.0`
  - Certified RHEL-based container images: `mcr.microsoft.com/mssql/rhel/server:vNext-CTP2.0`

- **Replication support (CTP 2.0)**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] supports SQL Server Replication on Linux. A Linux virtual machine with SQL Agent can be a publisher, distributor, or subscriber. 

  Create the following types of publications:
  - Transactional
  - Snapshot
  - Merge

  Configure replication [!INCLUDE[ssManStudioFull](../includes/ssmanstudiofull-md.md)] or use [replication stored procedures](../relational-databases/system-stored-procedures/replication-stored-procedures-transact-sql.md).

- **Support for the Microsoft Distributed Transaction Coordinator (MSDTC) (CTP 2.0)**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] on Linux supports the Microsoft Distributed Transaction Coordinator (MSDTC). For details, see [How to configure MSDTC on Linux](../linux/sql-server-linux-configure-msdtc.md).

- **OpenLDAP support for third-party AD providers (CTP 2.0)**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] on Linux supports OpenLDAP, which allows third-party providers to join Active Directory.

- **Machine Learning on Linux (CTP 2.0)**: [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Machine Learning Services (In-Database) is now supported on Linux. Support includes `sp_execute_external_script` stored procedure. For instructions on how to install Machine Learning Services on Linux, see [Install [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Machine Learning Services R and Python support on Linux](../linux/sql-server-linux-setup-machine-learning.md).

## <a id="mds"></a> Master Data Services 

- **Silverlight controls replaced with HTML (CTP 2.0)**: The Master Data Services (MDS) portal no longer depends on Silverlight. All the former Silverlight components have been replaced with HTML controls.

## <a id="security"></a>Security

- **Certificate management in SQL Server Configuration Manager (CTP 2.0)**: SSL/TLS certificates are widely used to secure access to SQL Server instances. Certificate management is now integrated into the SQL Server Configuration Manager, simplifying common tasks such as:

  - Viewing and validating certificates installed in a [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] instance. 
  - Viewing certificates close to expiration.
  - Deploy certificates across machines participating in Always On Availability Groups (from the node holding the primary replica).
  - Deploy certificates across machines participating in a failover cluster instance (from the active node).

  > [!NOTE]
  > User must have administrator permissions on all the cluster nodes.

## <a id="tools"></a>Tools

- [**Azure Data Studio**](../azure-data-studio/what-is.md): Previously released under the preview name SQL Operations Studio, Azure Data Studio is a lightweight, modern, open source, cross-platform desktop tool for the most common tasks in data development and administration. With Azure Data Studio and the [[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Preview extension](../azure-data-studio/sql-server-2019-extension.md) you can connect to SQL Server on premises and in the cloud on Windows, macOS, and Linux. Azure Data Studio allows you to:

  - AAD is now supported. (CTP 2.3)
  - Notebook view UI has moved into Azure Data Studio core. (CTP 2.3)
  - Added new wizard to create external data sources from Hadoop Distributed File System (HDFS) to SQL Server Big Data Cluster. (CTP 2.3)
  - Improved Notebook viewer UI. (CTP 2.3)
  - Added new Notebook APIs. (CTP 2.3)
  - Added "Reinstall Notebook dependencies" command to assist with Python package updates. (CTP 2.3)
  - Connect and manage [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] big data clusters. (CTP 2.1)
  - Edit and run queries in a modern development environment with lightning fast Intellisense, code snippets, and source control integration. (CTP 2.0) 
  - Quickly visualize data with built-in charting of your result sets. (CTP 2.0)
  - Create custom dashboards for your servers and databases using customizable widgets. (CTP 2.0)  
  - Easily manage your broader environment with the built-in terminal. (CTP 2.0)
  - Analyze data in an integrated notebook experience built on Jupyter. (CTP 2.0)
  - Enhance your experience with custom theming and extensions.(CTP 2.0)
  - And explore your Azure resources with a built-in subscription and resource browser. (CTP 2.0)
  - Supports scenarios using SQL Server big data cluster. (CTP 2.0)
  
  > [!TIP]
  > For the latest improvements to Azure Data Studio, see the [Azure Data Studio release notes](../azure-data-studio/release-notes-azure-data-studio.md).

- [**SQL Server Management Studio (SSMS) 18.0 (preview)**](../ssms/sql-server-management-studio-ssms.md): Supports [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)].

  - Launch Azure Data Studio from SSMS. (CTP 2.3)
  - Support for Always Encrypted with secure enclaves. (CTP 2.0)
  - Smaller download size. (CTP 2.0)
  - Now based on the Visual Studio 2017 Isolated Shell. (CTP 2.0)
  - For a complete list, see the [SSMS changelog](../ssms/release-notes-ssms.md). (CTP 2.0)

- [**SQL Server PowerShell module**](http://www.powershellgallery.com/packages/SqlServer/21.1.18080): The SqlServer PowerShell module allows SQL Server developers, admins, and BI professionals to automate database deployment and server administration.

  - Upgrade from 21.0 to 21.1 to support SMO v150.
  - Updated SQL Server provider (SQLRegistration) to display AS/IS/RS groups.
  - Fixed issue in `New-SqlAvailabilityGroup` cmdlet when targeting SQL Server 2014.
  - Added `–LoadBalancedReadOnlyRoutingList` parameter to `Set-SqlAvailabilityReplica` and `New-SqlAvailabilityReplica`.
  - Updated `AnalysisService` cmdlet to use cached login token from `Login-AzureAsAccount` for Azure Analysis Services.

## <a id="ssas"></a>[!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] Analysis Services (SSAS) 

### Many-to-many relationships in tabular models (CTP 2.4)

This feature allows many-to-many relationships between tables where both columns are non-unique. A relationship can be defined between a dimension and fact table at a granularity higher than the key column of the dimension. This avoids having to normalize dimension tables and can improve the user experience because the resulting model has a smaller number of tables with logically grouped columns. For this CTP 2.4 release, many-to-many relationships are engine-only features. 

Many-to-many relationships require models be at the 1470 compatibility level, which is currently supported only in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] CTP 2.3 and later. For this CTP 2.4 release, many-to-many relationships can be created by using the Tabular Object Model (TOM) API, Tabular Model Scripting Language (TMSL), and the open-source Tabular Editor tool. Support in SQL Server Data Tools (SSDT) will be included in a future release, as will documentation. Additional information for this and other CTP feature releases will be provided in the Analysis Services blog.

### Memory settings for resource governance (CTP 2.4)

The memory settings described here are already available in Azure Analysis Services. Beginning with CTP 2.4, they are now also supported by [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Analysis Services. 

- **Memory\QueryMemoryLimit** - This memory property can be used to limit memory spools built by DAX queries submitted to the model. 
- **DbpropMsmdRequestMemoryLimit** - This XMLA property can be used to override the Memory\QueryMemoryLimit server property value for a connection.
- **OLAP\Query\RowsetSerializationLimit** - This server property limits the number of rows returned in a rowset, protecting server resources from extensive data export usage. This property applies to both applies to both DAX and MDX queries.

These properties can be set by using the latest version of SQL Server Management Studio (SSMS). Additional information for this feature will be provided in the Analysis Services blog.

### Calculation groups in tabular models (CTP 2.3) 

Calculation groups address a common issue in complex models where there can be a proliferation of measures using the same calculations, such as time-intelligence. Calculation groups are shown in reporting clients as a table with a single column. Each value in the column represents a reusable calculation, or calculation item, that can be applied to any of the measures.  

A calculation group can have any number of calculation items. Each calculation item is defined by a DAX expression. Three new DAX functions are introduced to work with calculation groups: 

- `SELECTEDMEASURE()` - Returns a reference to the measure currently in context.  

- `SELECTEDMEASURENAME()` - Returns a string containing the name of the measure currently in context.  

- `ISSELECTEDMEASURE(M1, M2, …)` - Returns a Boolean indicating whether the measure currently in context is one of those specified as an argument.

In addition to new DAX functions, two new Dynamic Management Views are introduced:

- `TMSCHEMA_CALCULATION_GROUPS`  
- `TMSCHEMA_CALCULATION_ITEMS`  

#### Limitations in this release:

- The `ALLSELECTED DAX` function is not yet supported.
- Row Level Security defined on the calculation-group table is not yet supported.
- Object Level Security defined on the calculation-group table is not yet supported.
- DetailsRows expressions referring to calculation items are not yet supported.
- MDX is not yet supported.

#### Known issues in this release:

- The presence of calculation groups in a model may cause measures to return variant data types, which can cause refresh failures for calculated columns and tables that refer to measures.

#### Compatibility level

Calculation groups require models be at the 1470 compatibility level, which is currently supported only in [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] CTP 2.3 and later. At this time, calculation groups can be created by using the Tabular Object Model (TOM) API, Tabular Model Scripting Language (TMSL), and the open-source Tabular Editor tool. Support in SQL Server Data Tools (SSDT) will be included in a future release, as will documentation. Additional information for this and other CTP feature releases will be provided in the Analysis Services blog.

## Other services

As of CTP 2.4, [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] does not introduce new features for the following services:

- [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] [!INCLUDE[ssISnoversion](../includes/ssisnoversion-md.md)] (SSIS)
- [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] [!INCLUDE[ssRSnoversion](../includes/ssrsnoversion-md.md)] (SSRS)

## Next steps

- [[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] Release Notes](sql-server-ver15-release-notes.md).

- [Microsoft [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)]: Technical white paper](http://info.microsoft.com/rs/157-GQE-382/images/EN-US-CNTNT-white-paper-DBMod-Microsoft-SQL-Server-2019-Technical-white-paper.pdf)<br />Published in September 2018. Applies to Microsoft [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] CTP 2.0 for Windows, Linux, and Docker containers.

[!INCLUDE[get-help-options](../includes/paragraph-content/get-help-options.md)]
