# snowflake

## architecture

* storage
* compute (virtual warehouse): cache and micro-partitions
* cloud services: authentication, authorization, infrastructure manager, metadata manager, optimizer

## scalability

* vertical - depends on query dependency
* horizontal - driven my number of concurrent queries

## data loading

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
* 'CREATE STAGE' loads data from internal storage
* 'COPY' load data from STAGE to a table. Ingests data to snowflake in batches at time intervals. Snowflake 'COPY' command scheduled using Snowflake tasks or trigger copy commands using python/glue/airflow

COPY cammand can have:
* on error option
* validation option

#### transform data

Can be done with COPY command using SELECT statement

### loading unstructured data

You can load json, xml, parquet etc data formats including nested data, arrays etc..

To do that you need to add 'file format' to the COPY command.

Each document will be a new row and later you need to use 'dot notation', 'table' or 'flatten' functions.

### continuous data loading

* write / load the data into staging location (as before) on S3 and
* Snowpipe object (continous data ingestion) is triggered as soon as data is written to S3 and it will write the data to snowflake table

OR

* There is a Kafka-Snowflake connector

## Performance optimization

Traditional: add indexes, primary keys, partition data, analyze query plan

In snowflake: everything is managed in micro-partitions.

Optimization in snowflake:
* assign appropriate data types
* size virtual warehouse
* cluster keys: to locate micro-partitions. Managed by snowflake. Use it for very large (data size) tables.
* cache (store query results) 

## Snowpipe

Snowpipe is Snowflake continous data ingestion service. It loads data within minutes after files are added to a stage and submitted for ingestion. It loads data from staged files in mcro-batches (instead of COPY syayements that are used to load larger batches).

Steps:
* CREATE STAGE object
* CREATE PIPE object AS COPY INTO ...
* Create AWS S3 Event Notification on create events
* Create AWS SQS queue containing events

Snowpipe keeps the state of processing and it will not load the same file again.

Multiple files can be processed at the same time, but one file cannot be processed in parallel. File size should be 100-250MB. 

To expract data from Snowflake to AWS S3 use 'COPY' command

Automatically load new file if is loaded into some bucket / container. It is serverless.

You need to create 'STAGE', test 'COPY' command and create 'Notification' (to trigger it).

## Time Travel

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

## Table types

* __Permanent__: 'CREATE TABLE'. default, normal type wuth time travel and fail-safe
* __Transient__: 'CREATE TANSIENT TABLE'. No fail-safe. It is used where "data persistence" is required but doesn't need "data retention" for a longer period.
* __Temporary__: 'CREATE TEMPORARY TABLE'. No fail-safe. Exists only in __current session__ i.e. other users or sessions do not see it. Mostly used for transitory data like ETL/ELT
* __Dynamic__: 'CREATE DYNAMIC TABLE': Continously materlizes the results of the query you provide.

## View types

* Standard View
* Secure View: accessed only by authorized users
* Materialized View: These views store the result from the main source using filter conditions. Materialized view is auto-refreshed

## Zero-copy cloning

## Data Sharing

* data sharing without copy data
* shared data consumed by own compute

```
CREATE SHARE <share>
```

Reader account: non snowflake user. We pay for this account.

## Data Sampling

Take random sample of you data.

* Row method: every row has x% of being selected
* Block: every block has %x of being selected

## Streams

Objects that record changes made to a table. Follows CDC pattern: insert, update, delete statements are propagated.

```
CREATE STREAM <name> ON TABLE <table name>
SELECT * FROM <stream name>
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

## Change Tracking

Read-only alternative to streams. Enables querying change tracking metadata between two points in time

```
ALTER TABLE ... SET CHANGE_TRACKING = TRUE

SELECT * FROM ... AT ( TIMESTAMP > $ts1 )
```

Use it when you do not know when you will need to see or process changes.

## Tasks

Similar to scheduler. It can be executed as a single ot multiple SQL queries.

Tasks canbe dependent, but there can be only one parent.

```
CREATE TASK ... SCHEDULE ... AS ... query

OR

CREATE TASK ... AFTER ... AS ... query
```

## UDF = user defined functions

Reusable code written in sql, javascript, java, python

2 types of UDF:
* scalar: returns one output row
* tabular: returns 0,1, or many rows

```
CREATE FUNCTION funct1 (a number, b, number)
RETURNS number
language sql
AS
$$
    SELECT a+b
$$

SELECT funct1(1,2)
```

## Snowflake with Python and Spark

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


