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





