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

#### transform data

Can be done with COPY command using SELECT statement



### continuous data loading

Uses snowpipe