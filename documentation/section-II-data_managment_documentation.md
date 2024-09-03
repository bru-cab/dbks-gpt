### https://docs.databricks.com/en/delta/index.html
What is Delta Lake?
August 09, 2024

Delta Lake is the optimized storage layer that provides the foundation for tables in a lakehouse on Databricks. Delta Lake is open source software that extends Parquet data files with a file-based transaction log for ACID transactions and scalable metadata handling. Delta Lake is fully compatible with Apache Spark APIs, and was developed for tight integration with Structured Streaming, allowing you to easily use a single copy of data for both batch and streaming operations and providing incremental processing at scale.

Delta Lake is the default format for all operations on Databricks. Unless otherwise specified, all tables on Databricks are Delta tables. Databricks originally developed the Delta Lake protocol and continues to actively contribute to the open source project. Many of the optimizations and products in the Databricks platform build upon the guarantees provided by Apache Spark and Delta Lake. For information on optimizations on Databricks, see Optimization recommendations on Databricks.

For reference information on Delta Lake SQL commands, see Delta Lake statements.

The Delta Lake transaction log has a well-defined open protocol that can be used by any system to read the log. See Delta Transaction Log Protocol.

Getting started with Delta Lake
All tables on Databricks are Delta tables by default. Whether you’re using Apache Spark DataFrames or SQL, you get all the benefits of Delta Lake just by saving your data to the lakehouse with default settings.

For examples of basic Delta Lake operations such as creating tables, reading, writing, and updating data, see Tutorial: Delta Lake.

Databricks has many recommendations for best practices for Delta Lake.

Converting and ingesting data to Delta Lake
Databricks provides a number of products to accelerate and simplify loading data to your lakehouse.

Delta Live Tables:

Tutorial: Run your first ETL workload on Databricks

Load data using streaming tables (Python/SQL notebook)

Load data using streaming tables in Databricks SQL

COPY INTO

Auto Loader

Add data UI

Incrementally convert Parquet or Iceberg data to Delta Lake

One-time conversion of Parquet or Iceberg data to Delta Lake

Third-party partners

For a full list of ingestion options, see Ingest data into a Databricks lakehouse.

Updating and modifying Delta Lake tables
Atomic transactions with Delta Lake provide many options for updating data and metadata. Databricks recommends you avoid interacting directly with data and transaction log files in Delta Lake file directories to avoid corrupting your tables.

Delta Lake supports upserts using the merge operation. See Upsert into a Delta Lake table using merge.

Delta Lake provides numerous options for selective overwrites based on filters and partitions. See Selectively overwrite data with Delta Lake.

You can manually or automatically update your table schema without rewriting data. See Update Delta Lake table schema.

Enable columns mapping to rename or delete columns without rewriting data. See Rename and drop columns with Delta Lake column mapping.

Incremental and streaming workloads on Delta Lake
Delta Lake is optimized for Structured Streaming on Databricks. Delta Live Tables extends native capabilities with simplified infrastructure deployment, enhanced scaling, and managed data dependencies.

Delta table streaming reads and writes

Use Delta Lake change data feed on Databricks

Querying previous versions of a table
Each write to a Delta table creates a new table version. You can use the transaction log to review modifications to your table and query previous table versions. See Work with Delta Lake table history.

Delta Lake schema enhancements
Delta Lake validates schema on write, ensuring that all data written to a table matches the requirements you’ve set.

Schema enforcement

Constraints on Databricks

Delta Lake generated columns

Enrich Delta Lake tables with custom metadata

Managing files and indexing data with Delta Lake
Databricks sets many default parameters for Delta Lake that impact the size of data files and number of table versions that are retained in history. Delta Lake uses a combination of metadata parsing and physical data layout to reduce the number of files scanned to fulfill any query.

Use liquid clustering for Delta tables

Data skipping for Delta Lake

Optimize data file layout

Remove unused data files with vacuum

Configure Delta Lake to control data file size

Configuring and reviewing Delta Lake settings
Databricks stores all data and metadata for Delta Lake tables in cloud object storage. Many configurations can be set at either the table level or within the Spark session. You can review the details of the Delta table to discover what options are configured.

Review Delta Lake table details with describe detail

Delta table properties reference

Data pipelines using Delta Lake and Delta Live Tables
Databricks encourages users to leverage a medallion architecture to process data through a series of tables as data is cleaned and enriched. Delta Live Tables simplifies ETL workloads through optimized execution and automated infrastructure deployment and scaling.

Delta Lake feature compatibility
Not all Delta Lake features are in all versions of Databricks Runtime. For information about Delta Lake versioning, see How does Databricks manage Delta Lake feature compatibility?.

Delta Lake API documentation
For most read and write operations on Delta tables, you can use Spark SQL or Apache Spark DataFrame APIs.

For Delta Lake-spefic SQL statements, see Delta Lake statements.

Databricks ensures binary compatibility with Delta Lake APIs in Databricks Runtime. To view the Delta Lake API version packaged in each Databricks Runtime version, see the System environment section on the relevant article in the Databricks Runtime release notes. For documentation on Delta Lake APIs for Python, Scala, and Java, see the OSS Delta Lake documentation.


### https://docs.databricks.com/en/delta/history.html


Work with Delta Lake table history
July 26, 2024

Each operation that modifies a Delta Lake table creates a new table version. You can use history information to audit operations, rollback a table, or query a table at a specific point in time using time travel.

Note

Databricks does not recommend using Delta Lake table history as a long-term backup solution for data archival. Databricks recommends using only the past 7 days for time travel operations unless you have set both data and log retention configurations to a larger value.

Retrieve Delta table history
You can retrieve information including the operations, user, and timestamp for each write to a Delta table by running the history command. The operations are returned in reverse chronological order.

Table history retention is determined by the table setting delta.logRetentionDuration, which is 30 days by default.

Note

Time travel and table history are controlled by different retention thresholds. See What is Delta Lake time travel?.

Copy to clipboardCopy
SQL
DESCRIBE HISTORY table_name       -- get the full history of the table

DESCRIBE HISTORY table_name LIMIT 1  -- get the last operation only
For Spark SQL syntax details, see DESCRIBE HISTORY.

See the Delta Lake API documentation for Scala/Java/Python syntax details.

Catalog Explorer provides a visual view of this detailed table information and history for Delta tables. In addition to the table schema and sample data, you can click the History tab to see the table history that displays with DESCRIBE HISTORY.

What is Delta Lake time travel?
Delta Lake time travel supports querying previous table versions based on timestamp or table version (as recorded in the transaction log). You can use time travel for applications such as the following:

Re-creating analyses, reports, or outputs (for example, the output of a machine learning model). This could be useful for debugging or auditing, especially in regulated industries.

Writing complex temporal queries.

Fixing mistakes in your data.

Providing snapshot isolation for a set of queries for fast changing tables.

Important

Table versions accessible with time travel are determined by a combination of the retention threshold for transaction log files and the frequency and specified retention for VACUUM operations. If you run VACUUM daily with the default values, 7 days of data is available for time travel.

Delta time travel syntax
You query a Delta table with time travel by adding a clause after the table name specification.

timestamp_expression can be any one of:

'2018-10-18T22:15:12.013Z', that is, a string that can be cast to a timestamp

cast('2018-10-18 13:36:32 CEST' as timestamp)

'2018-10-18', that is, a date string

current_timestamp() - interval 12 hours

date_sub(current_date(), 1)

Any other expression that is or can be cast to a timestamp

version is a long value that can be obtained from the output of DESCRIBE HISTORY table_spec.

Neither timestamp_expression nor version can be subqueries.

Only date or timestamp strings are accepted. For example, "2019-01-01" and "2019-01-01T00:00:00.000Z". See the following code for example syntax:

SQL
Python
Copy to clipboardCopy
SELECT * FROM people10m TIMESTAMP AS OF '2018-10-18T22:15:12.013Z';
SELECT * FROM people10m VERSION AS OF 123;
You can also use the @ syntax to specify the timestamp or version as part of the table name. The timestamp must be in yyyyMMddHHmmssSSS format. You can specify a version after @ by prepending a v to the version. See the following code for example syntax:

SQL
Python
Copy to clipboardCopy
SELECT * FROM people10m@20190101000000000
SELECT * FROM people10m@v123
What are transaction log checkpoints?
Delta Lake records table versions as JSON files within the _delta_log directory, which is stored alongside table data. To optimize checkpoint querying, Delta Lake aggregates table versions to Parquet checkpoint files, preventing the need to read all JSON versions of table history. Databricks optimizes checkpointing frequency for data size and workload. Users should not need to interact with checkpoints directly. The checkpoint frequency is subject to change without notice.

Configure data retention for time travel queries
To query a previous table version, you must retain both the log and the data files for that version.

Data files are deleted when VACUUM runs against a table. Delta Lake manages log file removal automatically after checkpointing table versions.

Because most Delta tables have VACUUM run against them regularly, point-in-time queries should respect the retention threshold for VACUUM, which is 7 days by default.

In order to increase the data retention threshold for Delta tables, you must configure the following table properties:

delta.logRetentionDuration = "interval <interval>": controls how long the history for a table is kept. The default is interval 30 days.

delta.deletedFileRetentionDuration = "interval <interval>": determines the threshold VACUUM uses to remove data files no longer referenced in the current table version. The default is interval 7 days.

You can specify Delta properties during table creation or set them with an ALTER TABLE statement. See Delta table properties reference.

Note

You must set both of these properties to ensure table history is retained for longer duration for tables with frequent VACUUM operations. For example, to access 30 days of historical data, set delta.deletedFileRetentionDuration = "interval 30 days" (which matches the default setting for delta.logRetentionDuration).

Increasing data retention threshold can cause your storage costs to go up, as more data files are maintained.

Restore a Delta table to an earlier state
You can restore a Delta table to its earlier state by using the RESTORE command. A Delta table internally maintains historic versions of the table that enable it to be restored to an earlier state. A version corresponding to the earlier state or a timestamp of when the earlier state was created are supported as options by the RESTORE command.

Important

You can restore an already restored table.

You can restore a cloned table.

You must have MODIFY permission on the table being restored.

You cannot restore a table to an older version where the data files were deleted manually or by vacuum. Restoring to this version partially is still possible if spark.sql.files.ignoreMissingFiles is set to true.

The timestamp format for restoring to an earlier state is yyyy-MM-dd HH:mm:ss. Providing only a date(yyyy-MM-dd) string is also supported.

Copy to clipboardCopy
SQL
RESTORE TABLE target_table TO VERSION AS OF <version>;
RESTORE TABLE target_table TO TIMESTAMP AS OF <timestamp>;
For syntax details, see RESTORE.

Important

Restore is considered a data-changing operation. Delta Lake log entries added by the RESTORE command contain dataChange set to true. If there is a downstream application, such as a Structured streaming job that processes the updates to a Delta Lake table, the data change log entries added by the restore operation are considered as new data updates, and processing them may result in duplicate data.

For example:

Table version

Operation

Delta log updates

Records in data change log updates

0

INSERT

AddFile(/path/to/file-1, dataChange = true)

(name = Viktor, age = 29, (name = George, age = 55)

1

INSERT

AddFile(/path/to/file-2, dataChange = true)

(name = George, age = 39)

2

OPTIMIZE

AddFile(/path/to/file-3, dataChange = false), RemoveFile(/path/to/file-1), RemoveFile(/path/to/file-2)

(No records as Optimize compaction does not change the data in the table)

3

RESTORE(version=1)

RemoveFile(/path/to/file-3), AddFile(/path/to/file-1, dataChange = true), AddFile(/path/to/file-2, dataChange = true)

(name = Viktor, age = 29), (name = George, age = 55), (name = George, age = 39)

In the preceding example, the RESTORE command results in updates that were already seen when reading the Delta table version 0 and 1. If a streaming query was reading this table, then these files will be considered as newly added data and will be processed again.

Restore metrics
RESTORE reports the following metrics as a single row DataFrame once the operation is complete:

table_size_after_restore: The size of the table after restoring.

num_of_files_after_restore: The number of files in the table after restoring.

num_removed_files: Number of files removed (logically deleted) from the table.

num_restored_files: Number of files restored due to rolling back.

removed_files_size: Total size in bytes of the files that are removed from the table.

restored_files_size: Total size in bytes of the files that are restored.

Restore metrics example
Examples of using Delta Lake time travel
Fix accidental deletes to a table for the user 111:

Copy to clipboardCopy
SQL
INSERT INTO my_table
  SELECT * FROM my_table TIMESTAMP AS OF date_sub(current_date(), 1)
  WHERE userId = 111
Fix accidental incorrect updates to a table:

Copy to clipboardCopy
SQL
MERGE INTO my_table target
  USING my_table TIMESTAMP AS OF date_sub(current_date(), 1) source
  ON source.userId = target.userId
  WHEN MATCHED THEN UPDATE SET *
Query the number of new customers added over the last week.

Copy to clipboardCopy
SQL
SELECT count(distinct userId)
FROM my_table  - (
  SELECT count(distinct userId)
  FROM my_table TIMESTAMP AS OF date_sub(current_date(), 7))
How do I find the last commit’s version in the Spark session?
To get the version number of the last commit written by the current SparkSession across all threads and all tables, query the SQL configuration spark.databricks.delta.lastCommitVersionInSession.

SQL
Python
Scala
Copy to clipboardCopy
SET spark.databricks.delta.lastCommitVersionInSession
If no commits have been made by the SparkSession, querying the key returns an empty value.

Note

If you share the same SparkSession across multiple threads, it’s similar to sharing a variable across multiple threads; you may hit race conditions as the configuration value is updated concurrently.

### https://docs.databricks.com/en/tables/managed.html




Work with managed tables
August 13, 2024

Databricks manages the lifecycle and file layout for a managed table. Managed tables are the default way to create tables.

Databricks recommends that you use managed tables for all tabular data managed in Databricks.

Note

This article focuses on Unity Catalog managed tables. Managed tables in the legacy Hive metastore have different behaviors. See Database objects in the legacy Hive metastore.

Work with managed tables
You can work with managed tables across all languages and products supported in Databricks. You need certain privileges to create, update, delete, or query managed tables. See Manage privileges in Unity Catalog.

You should not use tools outside of Databricks to manipulate files in managed tables directly.

You should only interact with data files in a managed table using the table name.

Data files for managed tables are stored in the managed storage location associated with the containing schema. See Specify a managed storage location in Unity Catalog.

Create a managed table
By default, any time you create a table using SQL commands, Spark, or other tools in Databricks, the table is managed.

The following SQL syntax demonstrates how to create an empty managed table using SQL. Replace the placeholder values:

<catalog-name>: The name of the catalog that will contain the table.

<schema-name>: The name of the schema that will contain the table.

<table-name>: A name for the table.

<column-specification>: The name and data type for each column.

Copy to clipboardCopy
SQL
CREATE TABLE <catalog-name>.<schema-name>.<table-name>
(
  <column-specification>
);
Many users create managed tables from query results or DataFrame write operations. The following articles demonstrate some of the many patterns you can use to create a managed table on Databricks:

CREATE TABLE [USING]

CREATE TABLE LIKE

Create or modify a table using file upload

Required permissions
To create a managed table, you must have:

The USE SCHEMA permission on the table’s parent schema.

The USE CATALOG permission on the table’s parent catalog.

The CREATE TABLE permission on the table’s parent schema.

Drop a managed table
You must be the table’s owner to drop a table. To drop a managed table, run the following SQL command:

Copy to clipboardCopy
SQL
DROP TABLE IF EXISTS catalog_name.schema_name.table_name;
When a managed table is dropped, its underlying data is deleted from your cloud tenant within 30 days.

### https://docs.databricks.com/en/tables/external.html

Work with external tables
July 15, 2024

External tables store data in a directory in cloud object storage in your cloud tenant. You must specify a storage location when you define an external table.

Databricks recommends using external tables only when you require direct access to the data without using compute on Databricks. Unity Catalog privileges are not enforced when users access data files from external systems.

Note

This article focuses on Unity Catalog external tables. External tables in the legacy Hive metastore have different behaviors. See Database objects in the legacy Hive metastore.

Work with external tables
Databricks only manages the metadata for external tables and does not use the manage storage location associated with the containing schema. The table registration in Unity Catalog is just a pointer to data files. When you drop an external table, the data files are not deleted.

When you create an external table, you can either register an existing directory of data files as a table or provide a path to create new data files.

External tables can use the following file formats:

DELTA

CSV

JSON

AVRO

PARQUET

ORC

TEXT

Create an external table
To create an external table, can use SQL commands or Dataframe write operations.

Before you begin
To create an external table, you must meet the following permission requirements:

The CREATE EXTERNAL TABLE privilege on an external location that grants access to the LOCATION accessed by the external table.

The USE SCHEMA permission on the table’s parent schema.

The USE CATALOG permission on the table’s parent catalog.

The CREATE TABLE permission on the table’s parent schema.

For more information about configuring external locations, see Create an external location to connect cloud storage to Databricks.

Note

When you grant access to an external table, be aware of following: Databricks recommends that you grant write privileges on a table that is backed by an external location in S3 only if the external location is defined in a single metastore. You can safely read data in a single external S3 location from more than one metastore, but concurrent writes to the same S3 location from multiple metastores can lead to consistency issues.

SQL command examples
Use one of the following command examples in a notebook or the SQL query editor to create an external table.

In the following examples, replace the placeholder values:

<catalog>: The name of the catalog that will contain the table.

<schema>: The name of the schema that will contain the table.

<table-name>: A name for the table.

<column-specification>: The name and data type for each column.

<bucket-path>: The path to the cloud storage bucket where the table will be created.

<table-directory>: A directory where the table will be created. Use a unique directory for each table.

Copy to clipboardCopy
SQL
CREATE TABLE <catalog>.<schema>.<table-name>
(
  <column-specification>
)
LOCATION 's3://<bucket-path>/<table-directory>';
For more information about table creation parameters, see CREATE TABLE.

Dataframe write operations
Many users create external tables from query results or DataFrame write operations. The following articles demonstrate some of the many patterns you can use to create an external table on Databricks:

CREATE TABLE [USING]

CREATE TABLE LIKE

Drop an external table
To drop a table you must be its owner. To drop an external table, run the following SQL command:

Copy to clipboardCopy
SQL
DROP TABLE IF EXISTS catalog_name.schema_name.table_name;
Unity Catalog does not delete the underlying data in cloud storage when you drop an external table. You must directly delete the underlying data files if you need to remove data associated with the table.

### https://docs.databricks.com/en/views/index.html

What is a view?
June 27, 2024

A view is a read-only object composed from one or more tables and views in a metastore. A view can be created from tables and other views in multiple schemas and catalogs.

In Unity Catalog, views sit at the third level of the three-level namespace (catalog.schema.view):

Unity Catalog object model diagram, focused on view
This article describes the views that you can create in Databricks.

Views in Unity Catalog
A view stores the text of a query typically against one or more data sources or tables in the metastore. In Databricks, a view is equivalent to a Spark DataFrame persisted as an object in a schema. Unlike DataFrames, you can query views from anywhere in Databricks, assuming that you have permission to do so. Creating a view does not process or write any data. Only the query text is registered to the metastore in the associated schema.

Note

Views might have different execution semantics if they’re backed by data sources other than Delta tables. Databricks recommends that you always define views by referencing data sources using a table or view name. Defining views against datasets by specifying a path or URI can lead to confusing data governance requirements.

Materialized views
Materialized views incrementally calculate and update the results returned by the defining query.

You can register materialized views in Unity Catalog using Databricks SQL or define them as part of a Delta Live Tables pipeline. See Use materialized views in Databricks SQL and What is Delta Live Tables?.

Temporary views
A temporary view has limited scope and persistence and is not registered to a schema or catalog. The lifetime of a temporary view differs based on the environment you’re using:

In notebooks and jobs, temporary views are scoped to the notebook or script level. They cannot be referenced outside of the notebook in which they are declared, and no longer exist when the notebook detaches from the cluster.

In Databricks SQL, temporary views are scoped to the query level. Multiple statements within the same query can use the temp view, but it cannot be referenced in other queries, even within the same dashboard.

Dynamic views
Dynamic views can be used to provide row- and column-level access control, in addition to data masking. See Create a dynamic view.

Views in the Hive metastore (legacy)
You can define legacy Hive views against any data source and register them in the legacy Hive metastore. Databricks recommends migrating all legacy Hive views to Unity Catalog. See Views in Hive metastore.

Hive global temp view (legacy)
Global temp views are a legacy Databricks feature that allow you to register a temp view that is available to all workloads running against a compute resource. Global temp views are a legacy holdover of Hive and HDFS. Databricks recommends against using global temp views.

### https://docs.databricks.com/en/tables/index.html

What is a table?
July 26, 2024

A table resides in a schema and contains rows of data. All tables created in Databricks use Delta Lake by default. Tables backed by Delta Lake are also called Delta tables.

A Delta table stores data as a directory of files in cloud object storage and registers table metadata to the metastore within a catalog and schema. All Unity Catalog managed tables and streaming tables are Delta tables. Unity Catalog external tables can be Delta tables but are not required to be.

It is possible to create tables on Databricks that don’t use Delta Lake. These tables do not provide the transactional guarantees or optimized performance of Delta tables. You can choose to create the following tables types using formats other than Delta Lake:

External tables.

Foreign tables.

Tables registered to the legacy Hive metastore.

In Unity Catalog, tables sit at the third level of the three-level namespace (catalog.schema.table):

Unity Catalog object model diagram, focused on table
Databricks table types
Databricks enables you to use the following types of tables.

Managed tables
Managed tables manage underlying data files alongside the metastore registration. Databricks recommends that you use managed tables whenever you create a new table. Unity Catalog managed tables are the default when you create tables in Databricks. They always use Delta Lake. See Work with managed tables.

External tables
External tables, sometimes called unmanaged tables, decouple the management of underlying data files from metastore registration. Unity Catalog external tables can store data files using common formats readable by external systems. See Work with external tables.

Delta tables
The term Delta table is used to describe any table backed by Delta Lake. Because Delta tables are the default on Databricks, most references to tables are describing the behavior of Delta tables unless otherwise noted.

Databricks recommends that you always interact with Delta tables using fully-qualified table names rather than file paths.

Streaming tables
Streaming tables are Delta tables primarily used for processing incremental data. Most updates to streaming tables happen through refresh operations.

You can register streaming tables in Unity Catalog using Databricks SQL or define them as part of a Delta Live Tables pipeline. See Load data using streaming tables in Databricks SQL. and What is Delta Live Tables?.

Foreign tables
Foreign tables represent data stored in external systems connected to Databricks through Lakehouse Federation. Foreign tables are read-only on Databricks. See What is Lakehouse Federation?.

Feature tables
Any Delta table managed by Unity Catalog that has a primary key is a feature table. You can optionally configure feature tables using the online Feature Store for low-latency use cases. See Work with feature tables in workspace feature store.

Hive tables (legacy)
Hive tables describe two distinct concepts on Databricks, both of which are legacy patterns and not recommended.

Tables registered using the legacy Hive metastore store data in the legacy DBFS root, by default. Databricks recommends migrating all tables from the legacy HMS to Unity Catalog. See Database objects in the legacy Hive metastore.

Apache Spark supports registering and querying Hive tables, but these codecs are not optimized for Databricks. Databricks recommends registering Hive tables only to support queries against data written by external systems. See Hive table (legacy).

Live tables (deprecated)
The term live tables refers to an earlier implementation of functionality now implemented as materialized views. Any legacy code that references live tables should be updated to use syntax for materialized views. See What is Delta Live Tables? and Use materialized views in Databricks SQL.

Basic table permissions
To create a table, users must have CREATE TABLE and USE SCHEMA permissions on the schema, and they must have the USE CATALOG permission on its parent catalog. To query a table, users must have the SELECT permission on the table, the USE SCHEMA permission on its parent schema, and the USE CATALOG permission on its parent catalog.

For more on Unity Catalog permissions, see Manage privileges in Unity Catalog.

### https://docs.databricks.com/en/lakehouse-architecture/security-compliance-and-privacy/index.html

Security, compliance, and privacy for the data lakehouse
October 10, 2023

The architectural principles of the security, compliance, and privacy pillar are about protecting a Databricks application, customer workloads, and customer data from threats. As a starting point, the Databricks Security and Trust Center provides a good overview of the Databricks approach to security.

Security, compliance, and privacy lakehouse architecture diagram for Databricks.
Principles of security, compliance, and privacy
Manage identity and access using least privilege

The practice of identity and access management (IAM) helps you ensure that the right people can access the right resources. IAM addresses the following aspects of authentication and authorization: account management including provisioning, identity governance, authentication, access control (authorization), and identity federation.

Protect data in transit and at rest

Classify your data into sensitivity levels and use mechanisms such as encryption, tokenization, and access control where appropriate.

Secure your network and identify and protect endpoints

Secure your network and monitor and protect the network integrity of internal and external endpoints through security appliances or cloud services like firewalls.

Review the Shared Responsibility Model

Security and compliance are a shared responsibility between Databricks, the Databricks customer, and the cloud provider. It is important to understand which party is responsible for what part.

Meet compliance and data privacy requirements

You might have internal (or external) requirements that require you to control the data storage locations and processing. These requirements vary based on systems design objectives, industry regulatory concerns, national law, tax implications, and culture. Be mindful that you might need to obfuscate or redact personally identifiable information (PII) to meet your regulatory requirements. Where possible, automate your compliance efforts.

Monitor system security

Use automated tools to monitor your application and infrastructure. To scan your infrastructure for vulnerabilities and detect security incidents, use automated scanning in your continuous integration and continuous deployment (CI/CD) pipelines.