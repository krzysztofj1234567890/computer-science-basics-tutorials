# snowflake

## architecture

* storage
* compute (virtual warehouse)
* cloud services

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
* 'CREATE STAGE' loads data from internal storage
* 'COPY' load data from STAGE to a table

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
TODO

## Performance optimization

Traditional: add indexes, primary keys, partition data, analyze query plan

In snowflake: everything is managed in micro-partitions.

Optimization in snowflake:
* assign appropriate data types
* size virtual warehouse
* cluster keys: to locate micro-partitions. Managed by snowflake. Use it for very large (data size) tables.
* cache (store query results) 

## Snowpipe

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

* Permanent: 'CREATE TABLE'. default, normal type wuth time travel and fail-safe
* Transient: 'CREATE TANSIENT TABLE'. No fail-safe
* Temporary: 'CREATE TEMPORARY TABLE'. No fail-safe. Exists only in current session i.e. other users or sessions do not see it.

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




