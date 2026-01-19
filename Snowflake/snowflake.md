# snowflake

# Table of Contents
- [Architecture](#architecture)
- [Data Loading](#dataloading)
- [Performance Optimization](#performanceoptimization)
- [Snowpipe](#snowpipe)
- [Timetravel](#timetravel)
- [Table Types](#tabletypes)
- [View Types](#viewtypes)
- [Zero-copy cloning](#zerocopycloning)
- [Data Sharing](#datasharing)
- [Data Sampling](#datasampling)
- [Streams](#streams)
- [Change Tracking](#changetracking)
- [Tasks](#tasks)
- [UDF](#udf)
- [Stored Procedures](#storedprocedures)
- [Spark](#spark)


## Architecture <a id="architecture"></a>

* storage
* compute (virtual warehouse): cache and micro-partitions
* cloud services: authentication, authorization, infrastructure manager, metadata manager, optimizer

### Tools

* snowSight: Web UI
* snowSQL: command line interface
* SnowCLI: Comprehensive Command-Line Interface. extends the functionality of SnowSQL by: managing account configurations, setting up integrations, and automating complex tasks.
* SnowPipe: Continuous Data Ingestion
* snowPark:  Integration of Programming Languages

### scalability

* vertical - depends on query dependency
* horizontal - driven my number of concurrent queries

## Data Loading <a id="dataloading"></a>

* bulk
* continuous

### bulk data loading

Uses warehouse

#### stages

stage = database object containing location where we load data from

There are:

* external stage (aws s3 or azure blob storage).  
* internal stage: (local storage). 

#### load data to tables

Steps to load data:
* 'CREATE STORAGE INTEGRATION' object to be able to access your S3 bucket from Snowflake
* Create AWS Role to access the Snowflake Integration Object
* 'CREATE FILE FORMAT' in Snowflake
* 'CREATE STAGE' loads data from storage
```
CREATE STAGE my_s3_stage
  URL = 's3://mybucket/path/to/data/'
  CREDENTIALS = (AWS_KEY_ID = '<AWS_ACCESS_KEY>' 
                 AWS_SECRET_KEY = '<AWS_SECRET_KEY>')
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"');
```
* Create target table:
```
CREATE OR REPLACE TABLE my_table (
  id INT,
  name STRING,
  email STRING,
  created_at TIMESTAMP
);
```
* 'COPY' load data from STAGE to a table. Ingests data to snowflake in batches at time intervals. Snowflake 'COPY' command scheduled using Snowflake tasks or trigger copy commands using python/glue/airflow
```
COPY INTO my_table
  FROM @my_s3_stage
  FILES = ('data_file.csv')
  ON_ERROR = 'SKIP_FILE'        -- Skip the file if there is an error
  VALIDATION_MODE = 'RETURN_ERRORS'; -- Return errors in the output
```
* Monitor loading process:
```
SELECT * 
FROM INFORMATION_SCHEMA.COPY_HISTORY 
WHERE TABLE_NAME = 'MY_TABLE' 
ORDER BY LOAD_TIME DESC;
```

COPY command can have:
* on error option
* validation option

#### transform data

Can be done with COPY command using SELECT statement:

```
INSERT INTO transformed_data (user_id, full_name, age_category)
  SELECT 
    id,
    CONCAT(first_name, ' ', last_name) AS full_name,  -- Concatenating first and last name
    CASE 
      WHEN age < 18 THEN 'Minor'
      WHEN age BETWEEN 18 AND 65 THEN 'Adult'
      ELSE 'Senior'
    END AS age_category  -- Categorizing based on age
  FROM users;
```

### loading unstructured data

You can load json, xml, parquet etc data formats including nested data, arrays etc..

To do that you need to add 'file format' to the COPY command.

Each document will be a new row and later you need to use 'dot notation', 'table' or 'flatten' functions.

### continuous data loading

* write / load the data into staging location (as before) on S3 and
* Snowpipe object (continous data ingestion) is triggered as soon as data is written to S3 and it will write the data to snowflake table

OR

* There is a Kafka-Snowflake connector

#### Using Snowpipe for Continuous Data Loading

* Create an External Stage
```
CREATE STAGE my_s3_stage
  URL = 's3://mybucket/data/'
  CREDENTIALS = (AWS_KEY_ID = '<AWS_ACCESS_KEY>' 
                 AWS_SECRET_KEY = '<AWS_SECRET_KEY>')
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"');
```
* Create the Target Table
```
CREATE OR REPLACE TABLE my_table (
  id INT,
  name STRING,
  email STRING,
  created_at TIMESTAMP
);
```
* Create the Snowpipe
```
CREATE PIPE my_pipe
  AUTO_INGEST = TRUE
  AS
  COPY INTO my_table
  FROM @my_s3_stage
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"')
  ON_ERROR = 'SKIP_FILE';  -- Skips files with errors
```
* Set Up Event Notification on S3: To trigger Snowpipe automatically whenever a new file is uploaded to the S3 bucket
* Monitor the Data Loading Process
```
SELECT * 
FROM INFORMATION_SCHEMA.PIPE_USAGE_HISTORY
WHERE PIPE_NAME = 'MY_PIPE'
ORDER BY START_TIME DESC;
```


## Performance Optimization <a id="performanceoptimization"></a>

Traditional: add indexes, primary keys, partition data, analyze query plan

In snowflake: everything is managed in micro-partitions.

Optimization in snowflake:
* assign appropriate data types
* size virtual warehouse
* cluster keys: to locate micro-partitions. Managed by snowflake. Use it for very large (data size) tables.
* cache (store query results) 

### Data Modeling and Schema Design

- __Star Schema or Snowflake Schema__: Use a star or snowflake schema to structure your data efficiently. Fact tables store aggregated or transactional data, while dimension tables store descriptive attributes.
- __Avoid Large Fact Tables with Many Null Values__: Tables with many NULL values or sparse data increase storage costs and slow down queries. Normalize the data as much as possible to avoid large, sparse tables.
- Denormalization for Read Performance

### Query Optimization

Snowflake optimizes most queries automatically, but you can still make improvements by focusing on certain aspects of your queries.

- __Use JOIN Conditions Efficiently__: Avoid unnecessary joins and filter out data as early as possible in your query
- __Use WITH Clauses (CTE)__: Common Table Expressions (CTEs) can improve query readability and simplify complex queries, but avoid excessive use of them in very large datasets, as __it could create performance overhead__. Instead, consider using temporary tables or subqueries for complex operations.
- __Minimize the Use of DISTINCT and GROUP BY__
- Use __EXPLAIN__ to Analyze Query Plans

### Clustering and Partitioning

Clustering and partitioning are key for optimizing performance, especially when dealing with large datasets. Snowflake uses micro-partitioning by default, but you can optimize it further using clustering keys.

Clustering Keys:
- Define the columns that Snowflake should use to cluster the data at a physical level. This helps reduce the need for Snowflake to scan entire partitions during a query.
- If your queries often filter or group by a specific column, clustering by that column can speed up the query execution.

### Scaling the Virtual Warehouse

- __Scale Virtual Warehouses__: Snowflake allows you to scale virtual warehouses vertically (by increasing the size) or horizontally (by adding clusters).
- __Multi-Cluster Warehouses__: Use multi-cluster virtual warehouses to handle large concurrent workloads. Snowflake automatically scales the number of clusters based on the number of queries running


### Caching

Snowflake’s caching mechanism automatically stores the results of queries and the data in result cache and metadata cache.

### Data Compression

Snowflake uses columnar storage to compress data and improve I/O performance. Snowflake automatically applies compression algorithms like ZSTD, Snappy, and LZ4 for optimal storage and query performance.

### Monitoring and Query Profiling

Monitoring your queries, virtual warehouses, and storage is key to understanding where your performance bottlenecks lie.

- __Query Profiling__: Use Snowflake’s Query History and Query Profile tools to identify slow queries, unnecessary table scans, or missing indexes.
```
SELECT * 
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE QUERY_TEXT LIKE '%SELECT%'
ORDER BY START_TIME DESC;
```
- __Warehouse Monitoring__: Monitor warehouse usage and performance to ensure you're not over-provisioning or under-provisioning your compute resources.
- __Resource Monitoring__: Use Snowflake's Resource Monitors to track compute usage and prevent overages.


## Snowpipe <a id="snowpipe"></a>

Snowpipe is Snowflake continous data ingestion service. 
It loads data within minutes after files are added to a stage and submitted for ingestion. 
It loads data from staged files in micro-batches (instead of COPY statements that are used to load larger batches).

Steps:
* CREATE STAGE object
* CREATE PIPE object AS COPY INTO ...
* Create AWS S3 Event Notification on create events
* Create AWS SQS queue containing events

Snowpipe keeps the state of processing and it will not load the same file again.

Multiple files can be processed at the same time, but one file cannot be processed in parallel. File size should be 100-250MB. 

To extract data from Snowflake to AWS S3 use 'COPY' command

Automatically load new file if is loaded into some bucket / container. It is serverless.

You need to create 'STAGE', test 'COPY' command and create 'Notification' (to trigger it).



## Time Travel <a id="timetravel"></a>

Time Travel in Snowflake allows you to query, clone, or restore data as it existed at a specific point in time (__within a defined retention period__). 
This feature is very useful for __recovering lost data, auditing, and comparing historical versions of data__. 
Snowflake keeps a historical record of changes made to tables, which allows you to access data from the past without requiring backups.

Time Travel is built into Snowflake, and it works across different objects, including __tables, schemas, and databases__.

Snowflake tracks all changes to data in the system (__inserts, updates, deletes__) at the storage level. 
When a record is updated or deleted, __Snowflake doesn’t immediately remove it but instead marks it as changed__, and the original data is still available within the Time Travel window.

Go back 60 s

```
SELECT * FROM <table> AT (OFFSET => -60 )
```

Go back before date

```
SELECT * FROM <table> BEFORE (timestamp => '2023-11-12 ...'::timestamp)
```

To recreate table at an older monent, create a new table and reinsert all data

To undrop a dropped table:

```
UNDROP TABLE <table>
```

The default retention period for Time Travel in Snowflake is __1 to 90 days__, depending on your Snowflake edition and configuration.

You can set the __retention period for individual tables or databases__.

You can use Time Travel in Snowflake to access and revert to a previous table schema by querying historical versions of the table

Reverting to a Previous Schema:
```
UNDROP TABLE my_table BEFORE (STATEMENT => '2023-01-05 10:00:00');
```


## Table types <a id="tabletypes"></a>

* __Permanent__: 'CREATE TABLE'. default, normal type with time travel and fail-safe
* __Transient__: 'CREATE TANSIENT TABLE'. No fail-safe. It is used where "data persistence" is required but doesn't need "data retention" for a longer period.
* __Temporary__: 'CREATE TEMPORARY TABLE'. No fail-safe. Exists only in __current session__ i.e. other users or sessions do not see it. Mostly used for transitory data like ETL/ELT
* __Dynamic__: 'CREATE DYNAMIC TABLE': Continously materlizes the results of the query you provide.



## View types <a id="viewtypes"></a>

* __Standard View__
* __Secure View__: accessed only by authorized users
* __Materialized View__: These views store the result from the main source using filter conditions. Materialized view is auto-refreshed


## Zero-copy cloning <a id="zerocopycloning "></a>

Zero-copy cloning is a Snowflake feature that lets you create a copy of a database, schema, or table almost instantly without physically duplicating the data.

The clone initially __shares the same underlying storage (micro-partitions)__ as the source object, which makes it fast, storage-efficient, and cost-effective.

```
CREATE TABLE sales_clone
CLONE sales;
```


## Data Sharing <a id="datasharing"></a>

Data Sharing in Snowflake is a feature that allows one Snowflake account to share live, read-only access to its data with other Snowflake accounts without copying or moving the data.

* data sharing without copy data
* shared data consumed by own compute

```
CREATE SHARE <share>
```

Reader account: non snowflake user. We pay for this account.

### Example:

Provider: Create a Share
```
-- Create a share object
CREATE SHARE sales_share;

-- Add objects to share
GRANT USAGE ON DATABASE sales_db TO SHARE sales_share;
GRANT USAGE ON SCHEMA sales_db.public TO SHARE sales_share;
GRANT SELECT ON TABLE sales_db.public.orders TO SHARE sales_share;

-- Optional: add secure view for filtered access
GRANT SELECT ON VIEW sales_db.public.orders_secure TO SHARE sales_share;
```

Consumer: Access the Share:
```
-- Create a database from the share
CREATE DATABASE sales_db_from_partner
  FROM SHARE provider_account.sales_share;

-- Query the shared data
SELECT * FROM sales_db_from_partner.public.orders;
```


## Data Sampling <a id="datasampling"></a>

### Data samples

Take random sample of you data.

* Row method: every row has x% of being selected
* Block: every block has %x of being selected


Create data samples (random samples):
```
SELECT * 
FROM table_name
[SAMPLE (percentage | number_of_rows)];
```

### Synthetic Data

```
-- Generate 1000 rows of synthetic data
CREATE OR REPLACE TABLE test_data AS
SELECT
    SEQ4() AS id,
    RANDOM() AS value,
    CURRENT_DATE AS created_at
FROM TABLE(GENERATOR(ROWCOUNT => 1000));
```


## Streams <a id="streams"></a>

Streams are objects that record changes made to a table (or view). 
Follows CDC pattern: insert, update, delete statements are propagated.

Streams are most commonly __used together with Tasks__ to build continuous / incremental data pipelines inside Snowflake.

A stream does not store the changed data rows itself. It __only stores an offset__ (a pointer) in the table's version history.
When you query the stream → Snowflake computes the delta (what changed) since the last offset by looking at historical micro-partitions.


```
CREATE OR REPLACE STREAM <name> ON TABLE <table name>

# See what the stream currently contains
SELECT * FROM <stream name>

# Consume changes into the target table
-- One-time / scheduled MERGE (in real pipelines this goes inside a TASK)
MERGE INTO <target_table>> AS tgt
USING (
  SELECT
    customer_id,
    name,
    email,
    updated_at,
    -- Handle soft deletes if needed
    IFF(is_active = FALSE, 'DELETE', 'UPSERT') AS change_type
  FROM  <stream name>
  QUALIFY ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) = 1
) AS src
ON tgt.customer_id = src.customer_id
  AND tgt.is_current = TRUE

WHEN MATCHED AND src.change_type = 'DELETE' THEN
  UPDATE SET
    valid_to   = src.updated_at,
    is_current = FALSE

WHEN MATCHED AND src.change_type = 'UPSERT' THEN
  UPDATE SET
    valid_to   = src.updated_at,
    is_current = FALSE

WHEN MATCHED AND src.change_type = 'UPSERT' THEN
  INSERT (
    customer_id, name, email, valid_from, valid_to, is_current
  )
  VALUES (
    src.customer_id,
    src.name,
    src.email,
    src.updated_at,
    '9999-12-31',
    TRUE
  )

WHEN NOT MATCHED AND src.change_type = 'UPSERT' THEN
  INSERT (
    customer_id, name, email, valid_from, valid_to, is_current
  )
  VALUES (
    src.customer_id,
    src.name,
    src.email,
    src.updated_at,
    '9999-12-31',
    TRUE
  );
```

Types of streams:
* standard: tracks all DML
* append only: only inserts
* insert only: same as append-only but for external tables

CREATE STREAM ... ON TABLE ....


The 'stream' table contains the original table plus metadata (like if it was an update etc.)

Streams can contain transactions:
```
begin

multiple sql statements

commit
```


## Change Tracking <a id="changetracking"></a>

Read-only alternative to streams. Enables querying change tracking metadata between two points in time

```
ALTER TABLE ... SET CHANGE_TRACKING = TRUE

SELECT * FROM ... AT ( TIMESTAMP > $ts1 )
```

Use it when you do not know when you will need to see or process changes.



## Tasks    <a id="tasks"></a>

Snowflake Tasks are a built-in scheduling and orchestration feature used to automate SQL execution in Snowflake. 
They’re commonly used for ETL/ELT pipelines, data refreshes, and chaining transformations.

A Task is an object that:
- Runs SQL statements automatically
- Can run on a schedule (cron or interval) or
- Run after another task finishes (task dependencies / DAGs)
- Uses a warehouse or serverless compute

Tasks can be dependent, but there can be only one parent.

```
CREATE TASK ... SCHEDULE ... AS ... query
OR
CREATE TASK ... AFTER ... AS ... query
```

Example:
```
CREATE OR REPLACE TASK load_sales_task
  WAREHOUSE = compute_wh
  SCHEDULE = 'USING CRON 0 2 * * * UTC'
AS
INSERT INTO sales_final
SELECT *
FROM sales_stage;
```

Example: Task with Dependencies (DAG):
```
# root task
CREATE OR REPLACE TASK load_raw_task
  WAREHOUSE = compute_wh
  SCHEDULE = '10 MINUTE'
AS
COPY INTO raw_table
FROM @raw_stage;


# child task
CREATE OR REPLACE TASK transform_task
  WAREHOUSE = compute_wh
  AFTER load_raw_task
AS
INSERT INTO clean_table
SELECT *
FROM raw_table
WHERE valid = TRUE;

# final task
CREATE OR REPLACE TASK aggregate_task
  WAREHOUSE = compute_wh
  AFTER transform_task
AS
INSERT INTO agg_table
SELECT date, SUM(amount)
FROM clean_table
GROUP BY date;

# enable all tasks
ALTER TASK load_raw_task RESUME;

# check status
SHOW TASKS;

```

## UDF = user defined functions <a id="udf"></a>

A UDF is a named function that:
- Accepts input parameters
- Returns a single value
- Can be written in: SQL, JavaScript, Python (via Snowpark)

You can use UDFs in: SELECT, WHERE, JOIN, GROUP BY, ORDER BY

2 types of UDF:
* scalar: returns one output row
* tabular: returns 0,1, or many rows

SQL UDF Example:
```
CREATE FUNCTION funct1 (a number, b, number)
RETURNS number
language sql
AS
$$
    SELECT a+b
$$;

SELECT funct1(1,2)
```

Python UDF Example (Snowpark):
```
CREATE OR REPLACE FUNCTION normalize_text(txt STRING)
RETURNS STRING
LANGUAGE PYTHON
RUNTIME_VERSION = '3.8'
HANDLER = 'normalize'
AS
$$
def normalize(txt):
    return txt.lower().strip()
$$;
```

### UDF vs Stored Procedure

| Feature            | UDF                 | Stored Procedure              |
| ------------------ | ------------------- | ----------------------------- |
| Returns            | Single value        | Multiple values / result sets |
| DML allowed        | ❌ (SQL UDF)        | ✅                            |
| Called from SELECT | ✅                  | ❌                            |
| Control flow       | Limited             | Full                          |
| Best for           | Calculations, rules | ETL, workflows                |


## Snowflake Stored procedures <a id="storedprocedures"></a>

Snowflake Stored Procedures are used to execute procedural logic in Snowflake—things that go beyond a single SQL statement, such as loops, conditions, error handling, and multi-step workflows.

They are commonly used for ETL orchestration, data loading, auditing, and automation.

- Contains multiple SQL statements
- Supports control flow (IF, FOR, WHILE)
- Can execute DML and DDL
- Can return:
  - A single value, or
  - A table/result set
- Is written in: JavaScript, Python (Snowpark)
- Unlike UDFs, stored procedures cannot be used in SELECT statements.

Example:
```
CREATE OR REPLACE TABLE load_log (
    process_name STRING,
    status STRING,
    run_time TIMESTAMP
);

CREATE OR REPLACE PROCEDURE load_sales_data()
RETURNS STRING
LANGUAGE JAVASCRIPT
AS
$$
try {
    snowflake.execute({
        sqlText: `INSERT INTO sales_final
                  SELECT * FROM sales_stage`
    });
    snowflake.execute({
        sqlText: `INSERT INTO load_log
                  VALUES ('LOAD_SALES', 'SUCCESS', CURRENT_TIMESTAMP)`
    });
    return 'Load completed successfully';
} catch (err) {
    snowflake.execute({
        sqlText: `INSERT INTO load_log
                  VALUES ('LOAD_SALES', 'FAILED', CURRENT_TIMESTAMP)`
    });
    return 'Load failed: ' + err.message;
}
$$;

# call
CALL load_sales_data();
```

## Snowflake with Python and Spark <a id="spark"></a>

### Snowflake with pyton

Local environment:

```
import snowflake.connector
```

AWS Glue:
* Aws Glue assows you to run serverless spark. It will create a VM and after task done, it will stop VM.

 Deploy python job that connects to snowflake



## Best practices

Warehouse:
* set auto suspend (for ETL: immediately, SELECT queries: 10 min (use cache), DevOps/Data Science: 5 min)
* set auto resume
* set timeouts:
* set appropriate warehouse so you can process your queries

Table design:
* Staging table: transient (no need to time travel)
* Productive table: permanent
* Development tables: transient

Data types: use appropriete data types (timestamp, numbers etc.)

Set clusters only if necessary: large table, or query profile has lots of table scans

Monitor:
* usage of warehouse
* usage of storage

Retention period:
* staging tables or processing tables: do not need time travel
* production/curated tables: use time travel.
* large tables: expensive - maybe you do not need time travel

## Pricing

- fee for data storage
- fee for compute: function of warehouse size, number of clusters and time spent to execute queries

## Snowflake objects

- Account
- User
- Role
- Virtual Warehouse
- Resource Monitor
- Integration
- Database
- Schema
- Table
- View
- Stored Procedure
- User Defined Functions (UDF)
- Stage
- File Format
- Pipe
- Sequence

## Working with snowflake

### Load data to snowflake

#### Use snowsight UI web interface
- login to account
- create database
- create schema
- create table
- optional: create file format
- create stage
- import file to stage

#### Use snowSQL client

https://sivachandanc.medium.com/ingesting-local-files-to-snowflake-table-using-snowsql-396301578fde

- Connect to snowflake
```
snowsql -a <account_name> -u <username>
```

- Create a Table
```
use WAREHOUSE ??? ;
use DATABASE ?? ;
use SCHEMA pulic ;
CREATE TABLE ... ;
```

- load data to to default User stage
```
put 'file:////home/sivachandan/Downloads/MOCK_DATA.csv' @~/staged;

list @~;
```

- load data from stage to table
```
copy into <table>> from @~/staged/MOCK_DATA.csv.gz ;
```

#### load bulk data from azure

- load data into azure storage account

- CREATE STORAGE INTEGRATION

- Create Stage
```
CREATE STAGE azstage
URL = azure://<account>.blob.core.windows.net/<container>/<path>
CREDENTIALS=(AZURE_SAS_TOKEN=…)
```
- Copy data to table
```
COPY INTO <table>
FROM @azstage/newbatch
```


# Interview questions

## How to copy data from snowflake table to file on aws s3?

To copy data from a Snowflake table to a file in AWS S3, you can use the COPY INTO command

```
CREATE STAGE my_s3_stage
  URL = 's3://mybucket/snowflake-exports/'
  CREDENTIALS = (AWS_KEY_ID = '<AWS_ACCESS_KEY>' 
                 AWS_SECRET_KEY = '<AWS_SECRET_KEY>')
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"');
```

If you have a large amount of data, you may want to partition the files by a specific column (e.g., by order_date or region). This will create subdirectories in the S3 bucket for each partition, making the data more manageable.

```
COPY INTO @my_s3_stage/my_data_file_
  FROM (SELECT id, name, email, order_date FROM my_table)
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"')
  PARTITION_BY = (TO_CHAR(order_date, 'YYYY/MM/DD'))
  OVERWRITE = TRUE;
```

## What is SCD Type 1 or Type 2 or Type 3 etc.?

SCD stands for Slowly Changing Dimension — a set of techniques used in data warehousing to manage changes in dimension data over time.

### SCD Type 1

SCD Type 1 (also called "Overwrite" or "Replace") is the simplest and most common way to handle changes in dimension data in a data warehouse.

- You simply overwrite the old value with the new value
- No history is preserved
- The dimension table always shows only the current state

### SCD Type 2

SCD Type 2 (also called "Versioning" or "Add new row") is the most common Slowly Changing Dimension technique when you need to preserve history of changes in dimension attributes.

- You do NOT overwrite the existing row
- Instead, you add a completely new row with the updated values
- The old row is closed (marked as no longer current)
- Both old and new versions remain in the table → full history is preserved

Example:
| sk | customer_id | name       | email               | city       | valid_from          | valid_to            | is_current
|----|-------------|------------|---------------------|------------|---------------------|---------------------|-----------
| 1  | 1001        | Alice Chen | alice@gmail.com     | Piscataway | 2023-05-12 00:00:00 | 2025-06-15 00:00:00 | FALSE
| 2  | 1001        | Alice Chen | alice.new@proton.me | Piscataway | 2025-06-15 00:00:00 | 9999-12-31 23:59:59 | TRUE

- surrogate_key (sk): Unique identifier for each version1, 2, 3, ... (auto-increment or sequence)
- business_key (customer_id): Natural key from source system
- attributes: The actual dimension columns: name, email, city, department, etc
- valid_from: When this version became valid 
- valid_to: When this version stopped being valid
- is_current: Quick flag: is this the active version?

### SCD Type 3

SCD Type 3 (Slowly Changing Dimension Type 3) is a method used in data warehousing to track limited history of changes to dimension attributes — typically just the current value and the immediately previous value

Type 3 adds extra columns to the same row to store the historical value(s). This keeps the dimension table compact.

| CustomerID | CustomerName | Current_City | Previous_City | Current_State | Previous_State
|------------|--------------|--------------|---------------|---------------|------------------
| 1001       | Alice        | Seattle      | Portland      | WA            | OR
| 1002       | Bob          | Austin       | NULL          | TX            | NULL

Use it when:
- Attribute changes very rarely (1–2 times in the lifetime of the record)
- You only need the most recent change (or original value) for reporting
- You want to avoid row explosion / table growth (Type 2 problem)

### SCD Type 4

#### Kimball's Original / Official Type 4: Mini-Dimension

Ralph Kimball (in The Data Warehouse Toolkit) defines Type 4 as splitting rapidly or frequently changing attributes out of a large "monster" dimension into a separate mini-dimension table.

- Main dimension table → contains stable / slowly changing attributes + current value of the volatile ones (often updated as Type 1 = overwrite)
- Mini-dimension table → contains the frequently changing attributes + a surrogate key; new row added for each meaningful change (Type 2 style)
- Fact table → gets two foreign keys:
  - One to the main dimension (stable part)
  - One to the mini-dimension (captures the "as-of" profile at the time of the fact)

Advantages: History of volatile attributes preserved without exploding the main table

