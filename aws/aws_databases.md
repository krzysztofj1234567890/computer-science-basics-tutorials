# Table of Contents
- [RDS](#RDS)
- [RDS for PostgreSQL](#PostgreSQL)
- [Aurora](#Aurora)
- [DynamoDB](#DynamoDB)
- [Redshift](#Redshift)
- [Interview Questions](#InterviewQuestions)
# RDS <a id="RDS"></a>

Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the AWS Cloud. It provides cost-efficient, resizable capacity.

Advantages:
- manages backups, software patching, automatic failure detection, and recovery.
- high availability with a primary DB instance and a synchronous secondary DB instance that you can fail over to when problems occur. You can also use read replicas to increase read scaling.
- control access by using AWS Identity and Access Management (IAM) to define users and permissions.

Supports the following database engines:
- IBM Db2
- MariaDB
- Microsoft SQL Server
- MySQL
- Oracle Database
- PostgreSQL

You can run your DB instance in several Availability Zones, an option called a Multi-AZ deployment. When you choose this option, Amazon automatically provisions and maintains one or more secondary standby DB instances in a different AZ. 

## RDS monitoring

- __DB instance status and recommendations__: View details about the current status of your instance by using the Amazon RDS console, AWS CLI, or RDS API. You can also respond to automated recommendations for database resources, such as DB instances, read replicas, and DB parameter groups
- __CloudWatch metrics__ for Amazon RDS:: Use it to monitor the performance and health of a DB instance. CloudWatch performance charts are shown in the Amazon RDS console. Amazon RDS automatically sends metrics to CloudWatch every minute for each active database. You don't get additional charges for Amazon RDS metrics in CloudWatch.
- __Performance Insights and operating-system monitoring__: assess the load on your database, and determine when and where to take action.
- __Integrated AWS services__: Amazon RDS is integrated with Amazon EventBridge, Amazon CloudWatch Logs, and Amazon DevOps Guru.

## RDS Setup

- Create a user with administrative access
- Sign in as the user with administrative access
- Assign access to additional users
- Determine requirements: specify details like storage, memory, database engine and version, network configuration, security, and maintenance periods
- VPC, subnet, and security group – Your DB instance will most likely be in a virtual private cloud (VPC). To connect to your DB instance, you need to set up security group rules.
- High availability – Do you need failover support? On Amazon RDS, a Multi-AZ deployment creates a primary DB instance and a secondary standby DB instance in another Availability Zone for failover support. 
- Provide access to your DB instance in your VPC by creating a security group

## RDS Proxy

You can set up the connection between your Lambda function and your DB instance through RDS Proxy to improve your database performance and resiliency. Often, Lambda functions make frequent, short database connections that benefit from connection pooling that RDS Proxy offers.

# RDS for PostgreSQL <a id="PostgreSQL"></a>

## Creating and connecting to a PostgreSQL DB instance

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html

Create:
- Create an EC2 instance - db Client
  - Name: ec2-database-connect
  - Key pair (login): choose a Key pair name to use an existing key pair or create a new one
  - Allow SSH traffic in Network settings: choose the source of SSH connections to the EC2 instance. You can choose My IP
- Create a PostgreSQL DB instance
  - Open the Amazon RDS console at https://console.aws.amazon.com/rds/.
  - In Configuration, choose PostgreSQL.
  - For DB instance size, choose Free tier.
  - For DB instance identifier, enter database-test1.
  - For Master username, enter a name for the master user,
  - select Auto generate a password.
  - To set up a connection with the EC2 instance you created previously, open Set up EC2 connection - optional.
    - Select Connect to an EC2 compute resource. 
  - Choose Create database

Connect to a PostgreSQL DB instance
- On database setup find out the endpoint and the port number to connect to the DB instance. 
- Connect to the EC2 instance (db client) that you created earlier
```
ssh -i location_of_pem_file ec2-user@ec2-instance-public-dns-name
```
- Install the psql command-line client from PostgreSQL on Amazon Linux 2023, run the following command:
```
sudo dnf install postgresql15
```
- Connect to the PostgreSQL DB instance:
```
psql --host=endpoint --port=5432 --dbname=postgres --username=postgres
```

## Best practices

- Use metrics to monitor your memory, CPU, replica lag, and storage usage ( Amazon CloudWatch)
- Scale up your DB instance when you are approaching storage capacity limits
- Enable automatic backups
- If your client application is caching the Domain Name Service (DNS) data of your DB instances, set a time-to-live (TTL) value of less than 30 seconds. The underlying IP address of a DB instance can change after a failover. Caching the DNS data for an extended time can thus lead to connection failures. 
- Test failover for your DB instance to understand how long the process takes
- Memory: allocate enough RAM so that your working set resides almost completely in memory. The working set is the data and indexes that are frequently in use on your instance.
  To tell if your working set is almost all in memory, check the ReadIOPS metric (using Amazon CloudWatch) while the DB instance is under load. 
  The value of ReadIOPS should be small and stable.
- AWS database drivers: AWS suite of drivers for application connectivity. The drivers have been designed to provide support for faster switchover and failover times, and authentication with AWS Secrets Manager, AWS Identity and Access Management (IAM), and Federated Identity
- Basline: To troubleshoot performance issues, it's important to understand the baseline performance of the system. When you set up a DB instance and run it with a typical workload, capture the average, maximum, and minimum values of all performance metrics. Do so at a number of different intervals (for example, one hour, 24 hours, one week, two weeks). This can give you an idea of what is normal. 
- Replica lag: If you use Multi-AZ DB clusters, monitor the time difference between the latest transaction on the writer DB instance and the latest applied transaction on a reader DB instance. This difference is called replica lag

To view performance metrics:
- Open RDS console
- DB instance -> monitoring

Analyzing database performance:

# Aurora <a id="Aurora"></a>

Amazon Aurora (Aurora) is a fully managed relational database engine that's compatible with MySQL and PostgreSQL.

The speed and reliability of high-end commercial databases with the simplicity and cost-effectiveness of open-source databases. 
The code, tools, and applications you use today with your existing MySQL and PostgreSQL databases can be used with Aurora. 
With some workloads, Aurora can deliver up to five times the throughput of MySQL and up to three times the throughput of PostgreSQL without requiring changes

Aurora includes a high-performance storage subsystem. The underlying storage grows automatically as needed.

## Aurora DB clusters

An Amazon Aurora DB cluster consists of one or more DB instances and a cluster volume that manages the data for those DB instances. 
An Aurora cluster volume is a virtual database storage volume that spans multiple Availability Zones, with each Availability Zone having a copy of the DB cluster data. 

Two types of DB instances make up an Aurora DB cluster:
- Primary (writer) DB instance – Supports read and write operations, and performs all of the data modifications to the cluster volume. Each Aurora DB cluster has one primary DB instance.
- Aurora Replica (reader DB instance) – Connects to the same storage volume as the primary DB instance but supports only read operations. Each Aurora DB cluster can have up to 15 Aurora Replicas

## Aurora Features

- blue-green deployments: By using Amazon RDS Blue/Green Deployments, you can make changes to the database in the staging environment without affecting the production environment. 
- export data to S3: You can export Aurora DB cluster data to an Amazon S3 bucket. After the data is exported, you can analyze the exported data directly through tools like Amazon Athena or Amazon Redshift Spectrum
- global databases: An Aurora global database is a single database that spans multiple AWS Regions, enabling low-latency global reads and disaster recovery from any Region-wide outage
- zero-ETL: Amazon Aurora zero-ETL integrations with Amazon Redshift is a fully managed solution for making transactional data available in Amazon Redshift after it's written to an Aurora cluster.
- RDS proxy: Amazon RDS Proxy is a fully managed, highly available database proxy that makes applications more scalable by pooling and sharing established database connections.
- Serverless: Aurora Serverless v2 is an on-demand, auto-scaling feature designed to be a cost-effective approach to running intermittent or unpredictable workloads on Amazon Aurora. It automatically scales capacity up or down as needed by your applications.

## Aurora global databases

An Aurora global database consists of one primary AWS Region where your data is written, and up to five read-only secondary AWS Regions. 
You issue write operations directly to the primary DB cluster in the primary AWS Region. 
Aurora replicates data to the secondary AWS Regions using dedicated infrastructure, with latency typically under a second.

## RDS Proxy

- By using Amazon RDS Proxy, you can allow your applications to __pool and share database connections__ to improve their ability to __scale__. 
- RDS Proxy makes applications more __resilient__ to database failures by __automatically connecting to a standby DB instance while preserving application connections__. 
- By using RDS Proxy, you can also enforce __AWS Identity and Access Management__ (IAM) __authentication__ for databases, and securely store credentials in AWS Secrets Manager. 

RDS Proxy __queues__ or __throttles__ application connections that can't be served immediately from the connection pool. 
Although latencies might increase, your application can continue to scale without abruptly failing or overwhelming the database

## Aurora zero-ETL integrations with Amazon Redshift

Enables near real-time analytics and machine learning (ML) using Amazon Redshift on petabytes of transactional data from Aurora

The source DB cluster must be in the same Region as the target Amazon Redshift data warehouse.

## Aurora Serverless v2

On-demand, autoscaling configuration for Amazon Aurora

Capacity is adjusted automatically based on application demand. 
You're charged only for the resources that your DB clusters consume.

## Aurora vs RDS

https://aws.amazon.com/blogs/database/is-amazon-rds-for-postgresql-or-amazon-aurora-postgresql-a-better-choice-for-me/

### storage

Aurora PostgreSQL uses a __high-performance storage subsystem__ customized to take advantage of __fast distributed storage__. 
The underlying storage __grows automatically in segments of 10 GiB, up to 128 TiB__. 
Aurora __improves__ upon PostgreSQL for __massive throughput and highly concurrent workloads__

Aurora PostgreSQL uses a single, virtual cluster volume that is supported by storage nodes using locally attached SSDs. 
A cluster volume consists of copies of the data across multiple Availability Zones in a single AWS Region. 
Aurora storage automatically increases the size of the database volume as the database storage grows.

Because the data is __automatically replicated across three Availability Zones__, data is highly available and durable.

vs

RDS for PostgreSQL supports up to __64 TiB of storage__ and recent PostgreSQL versions. 
DB instances for Amazon RDS for PostgreSQL use Amazon Elastic Block Store (Amazon __EBS__) volumes for database and log storage. 
RDS for PostgreSQL manages PostgreSQL installation, upgrades, storage management, replication for high availability, and backups for disaster recovery.

In addition to classic Multi-AZ configuration with single standby instance, RDS for PostgreSQL also supports Multi-AZ DB cluster. 
A Multi-AZ DB cluster is a semi-synchronous, highly available configuration with two readable standby DB instances. 
A Multi-AZ DB cluster consists a writer DB instance and two reader DB instances in three separate Availability Zones.

Amazon RDS for PostgreSQL supports Amazon EBS solid state drive (SSD)

General Purpose SSD gp2 storage delivers a consistent baseline of 3 IOPS per provisioned GiB and can burst up to 3,000 IOPS

Amazon RDS for PostgreSQL supports storage auto scaling. This feature automatically increases DB instance storage size in chunks of 10 GiB, or 10% of the currently allocated storage

### backups

Aurora PostgreSQL backs up DB cluster volume automatically and retains backups for the length of the defined retention period. 
Aurora automated backups are continuous and incremental. 
Restore time depends on the volume size and number of transactions logged that need to be restored. 
There is no performance impact or interruption of database service during backups

vs

Amazon RDS automatically takes daily backups of PostgreSQL DB instances one time during a backup window. 
There is a slight performance impact when the backup initiates for single Availability Zone deployments. 
In addition, it also continuously archives transaction logs (WALs)

### Scalability

Amazon Aurora readers and Amazon RDS read replicas help reduce the load on the primary DB instance by offloading read workloads to the readers/replicas.

Aurora PostgreSQL supports up to 15 readers for scaling out read workloads and high availability within a single AWS Region. 
Aurora provides this by scaling storage across three Availability Zones in the AWS Region. 
It writes the log records to six copies in three Availability Zones. 
Since Aurora uses shared storage for writer and readers, the impact of high write workloads on replication is negligible. 
All Aurora readers are synced with the writer DB instance with minimal replica lag.

Usually, this replica lag is a few hundred milliseconds

vs

With RDS for PostgreSQL, you can create three levels of cascaded read replicas, 5 replicas per instance up to total 155 read replicas per source instance.
Cascading read replicas help scale reads without adding overhead to source PostgreSQL DB instance.

ou can also promote read replicas when needed to become standalone DB instances. 
RDS for PostgreSQL also supports five cross-Region read replicas. 
Replicas are synced with the source DB instance using PostgreSQL streaming replication. 

 High write activity at source DB instance, storage type mismatch, and DB instance class mismatch can cause high replication lag. 
 This lag can be up to several minutes. 
 With optimal configurations and workload, in Amazon RDS for PostgreSQL the replica lag is typically a few seconds. 

### Failover

Multi-AZ Aurora PostgreSQL, the failover time is typically within __30 seconds__, which consists of DNS propagation, and recovery

VS 

Amazon RDS automatically detects a problem with primary database instance and triggers a failover. 
In the case of failover, read/write connections are automatically redirected to the promoted primary instance.

In Amazon RDS for Multi-AZ PostgreSQL, the failover time is typically around __1-2 minutes__


### Summary

Amazon Aurora advantages include:
- Offers up to __5X better__ performance than conventional MySQL databases and up to 3X better than PostgreSQL DBs.
- Aurora delivers up to __3X read replicas__ than Amazon RDS.
- The service delivers __low-latency read replicas__ across multiple Availability Zones in an AWS Region.
- Amazon Aurora delivers outstanding automated backups and supports__ data recoveries up to the last five minutes__.
- The Aurora architecture makes it faster, more durable, and cloud-native than RDS on Amazon EC2.
- Aurora __Serverless__ is highly performant and cost-effective for unpredictable database workloads.

Amazon Aurora limitations include:
- It is compatible with __just two DB engines__ (MySQL and PostgreSQL) compared to seven on RDS on Amazon EC2.
- Aurora is limited to the __InnoDB storage engine__.
- While Amazon RDS enables you to try it out for a year on the AWS Free Tier, there’s no such offer for Aurora.
- It’s t__ough to predict Amazon Aurora Serverless costs in advance__.

# DynamoDB <a id="DynamoDB"></a>

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html

Amazon DynamoDB is a serverless, NoSQL, fully managed database with single-digit millisecond performance at any scale.

DynamoDB addresses your needs to overcome scaling and operational complexities of relational databases. 
DynamoDB is purpose-built and optimized for operational workloads that require consistent performance at any scale. 

DynamoDB scales to support tables of virtually any size while providing consistent single-digit millisecond performance and high availability. 

Features:
- __Serverless__: With DynamoDB, you don't need to provision any servers, or patch, manage, install, maintain, or operate any software. DynamoDB provides zero downtime maintenance. It has no versions
- __NoSQL__: DynamoDB is purpose-built to deliver improved performance, scalability, manageability, and flexibility compared to traditional relational databases. 
To support a wide variety of use cases, DynamoDB supports both key-value and document data models.
- __Fully managed__
- __Single-digit millisecond performance at any scale__
- __ACID__: You can use DynamoDB transactions to achieve atomicity, consistency, isolation, and durability (ACID) across one or more tables with a single request.
- __Multi-active replication__: __Global tables__ provide multi-active replication of your data across your __chosen AWS Regions with 99.999% availability.__ 
Global tables deliver a fully managed solution for deploying a multi-Region, multi-active database, without building and maintaining your own replication solution. 
- __Change data capture__: DynamoDB supports streaming of item-level change data capture (CDC) records in near-real time. 
It offers two streaming models for CDC: __DynamoDB Streams__ and Kinesis Data Streams for DynamoDB
- __Secondary indexes__: DynamoDB offers the option to create both global and local secondary indexes, which let you query the table data using an alternate key.
- __S3 integration__: Integrating DynamoDB with Amazon S3 enables you to easily export data to an Amazon S3 bucket for analytics and machine learning
- __Zero-ETL integration__: DynamoDB supports zero-ETL integration with Amazon Redshift and Amazon OpenSearch Service. 
These integrations enable you to run complex analytics and use advanced search capabilities on your DynamoDB table data.
- __DAX (Caching)__: is a fully managed, highly available caching service built for DynamoDB. DAX delivers up to 10 times performance improvement – from milliseconds to microseconds – even at millions of requests per second. DAX does all the heavy lifting required to add in-memory acceleration to your DynamoDB tables, without requiring you to manage cache invalidation, data population, or cluster management.
- Resilience: By default, DynamoDB automatically replicates your data across three Availability Zones to provide high durability and a 99.99% availability SLA.

## NoSQL workbench for DynamoDB

NoSQL Workbench is a visual development tool that provides data modeling, data visualization, and query development features to help you design, create, query, and manage DynamoDB tables.

NoSQL Workbench now includes DynamoDB local as an optional part of the installation process, which makes it easier to model your data in DynamoDB local

Features:
- Data modeling: build new data models from, or design models based on, existing data models that satisfy your application's data access patterns. You can also import and export the designed data model at the end of the process
- Data visualization: map queries and visualize the access patterns (facets) of the application without having to write code.
- Operation building: rich graphical user interface for you to develop and test queries.

## Setup and Example

### Setup

- setup webservice on AWS
- Download DynamoDB local
- Run DynamoDB local as Docker image  - docker-compose.yaml:
```
version: '3.8'
services:
 dynamodb-local:
   command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
   image: "amazon/dynamodb-local:latest"
   container_name: dynamodb-local
   ports:
     - "8000:8000"
   volumes:
     - "./docker/dynamodb:/home/dynamodblocal/data"
   working_dir: /home/dynamodblocal
```

### Usage Example

Create table:
```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD
```

Write data:
```
aws dynamodb put-item \
    --table-name Music  \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}, "Awards": {"N": "1"}}'
```

Read data:
```
aws dynamodb get-item --consistent-read \
    --table-name Music \
    --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}'
```

Query:
```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :name" \
    --expression-attribute-values  '{":name":{"S":"Acme Band"}}'
```

## Concepts

Read capacity unit (__RCU__) – One strongly consistent read per second, or two eventually consistent reads per second, for items up to 4 KB in size.

Write capacity unit (__WCU__) – One write per second, for items up to 1 KB in size.

__Page size limit for query and scan__ – There is a limit of 1 MB per page, per query or scan. If your query parameters or scan operation on a table result in more than 1 MB of data, DynamoDB returns the initial matching items. It also returns a LastEvaluatedKey property that you can use in a new request to read the next page.

Local secondary indexes (__LSIs__) – You can define a maximum of five local secondary indexes.

Global secondary indexes (__GSIs__) – There is a default quota of 20 global secondary indexes per table

A __table__ is a collection of items, and each __item__ is a collection of __attributes__. Table is __schemaless__, which means that neither the attributes nor their data types need to be defined beforehand. Each item can have its own distinct attributes.  DynamoDB supports __nested attributes__ up to 32 levels deep.

DynamoDB uses __primary keys to uniquely identify each item__ in a table and __secondary indexes__ to provide more querying flexibility.

When you create a table, in addition to the table name, you must specify the __primary key__ of the table. The __primary key uniquely identifies each item in the table__, so that no two items can have the same key.

DynamoDB supports two different kinds of primary keys:
- __Partition key__ – A simple primary key, composed of one attribute known as the partition key. DynamoDB uses the partition key's value as input to an internal hash function.
- Partition key and sort key – Referred to as a __composite primary key__, this type of key is composed of two attributes. The first attribute is the partition key, and the second attribute is the sort key.  All items with the same partition key value are stored together, in sorted order by sort key value. In a table that has a partition key and a sort key, it's possible for multiple items to have the same partition key value. However, those items must have different sort key values.

DynamoDB supports two kinds of indexes:
- __Global secondary index__ – An index with a partition key and sort key that can be different from those on the table.
- __Local secondary index__ – An index that has the same partition key as the table, but a different sort key.

__table classes__:
-  DynamoDB Standard table class is the default. 
-  DynamoDB Standard-Infrequent Access (DynamoDB Standard-IA) table class is optimized for tables where storage is the dominant cost. For example, tables that store infrequently accessed data, such as application logs, old social media posts, e-commerce order history, 

Amazon DynamoDB stores data in partitions. A __partition__ is an allocation of storage for a table, backed by solid state drives (SSDs) and automatically replicated across multiple Availability Zones within an AWS Region. Partition management is handled entirely by DynamoDB—you never have to manage partitions yourself. DynamoDB uses the value of the partition key as input to an internal hash function. The output value from the hash function determines the partition in which the item will be stored. tends to keep items which have the same value of partition key close together and in sorted order by the sort key attribute's value.

### expression attribute values

Expression attribute values in Amazon DynamoDB act as variables. They're substitutes for the actual values that you want to compare—values that you might not know until runtime. An expression attribute value must begin with a colon (:) and be followed by one or more alphanumeric characters.

An expression attribute name is an alias (or placeholder) that you use in an Amazon DynamoDB expression as an alternative to an actual attribute name. 

```
aws dynamodb get-item \
    --table-name ProductCatalog \
    --key '{"Id":{"N":"123"}}' \
    --projection-expression "#sw" \
    --expression-attribute-names '{"#sw":"Safety.Warning"}'
```

### query vs scan

#### query
You must provide the name of the partition key attribute and a single value for that attribute. Query returns all items with that partition key value. Optionally, you can provide a sort key attribute and use a comparison operator to refine the search results.

```
aws dynamodb query \
    --table-name Thread \
    --key-condition-expression "ForumName = :name and Subject = :sub" \
    --expression-attribute-values  file://values.json
```

#### scan

A Scan operation in Amazon DynamoDB __reads every item in a table or a secondary index__. By default, a Scan operation returns all of the data attributes for every item in the table or index. You can use the ProjectionExpression parameter so that Scan only returns some of the attributes, rather than all of them.

Scan always returns a result set. If no matching items are found, the result set is empty.

A single Scan request can retrieve a maximum of 1 MB of data. 

Scan, as the name suggests, will browse table items from start to finish. The sort-key allows to determine the scanning order direction.

It is possible to apply filtersto both a Query and a Scan operation and control which items are returned. Filters do not contribute to optimizing the operation. They are applied after the operation execution and before results are returned.

Running a Scan is expensive and inefficient, thus should be avoided in almost all use cases. Unless one really needs to browse the entire set of items, for instance.

Querying by the primary-key is the most efficient manner of retrieving data from a DynamoDB table.

### PartiQL

PartiQL provides SQL-compatible query access across multiple data stores containing structured data, semistructured data, and nested data. It is widely used within Amazon and is now available as part of many AWS services, including DynamoDB.

### Dynamo Streams

Captures data modification events in DynamoDB tables.

Each event is represented by a stream record. If you enable a stream on a table, DynamoDB Streams writes a stream record whenever one of the following events occurs:
- A new item is __added__ to the table: The stream captures an image of the entire item, including all of its attributes.
- An item is __updated__: The stream captures the "before" and "after" image of any attributes that were modified in the item.
- An item is __deleted__

You can use DynamoDB Streams together with __AWS Lambda to create a trigger—code__ that runs automatically whenever an event of interest appears in a stream.

```
'use strict';
var AWS = require("aws-sdk");
var sns = new AWS.SNS();

exports.handler = (event, context, callback) => {

    event.Records.forEach((record) => {
        console.log('Stream record: ', JSON.stringify(record, null, 2));

        if (record.eventName == 'INSERT') {
            var who = JSON.stringify(record.dynamodb.NewImage.Username.S);
            var when = JSON.stringify(record.dynamodb.NewImage.Timestamp.S);
            var what = JSON.stringify(record.dynamodb.NewImage.Message.S);
            var params = {
                Subject: 'A new bark from ' + who,
                Message: 'Woofer user ' + who + ' barked the following at ' + when + ':\n\n ' + what,
                TopicArn: 'arn:aws:sns:region:accountID:wooferTopic'
            };
            sns.publish(params, function(err, data) {
                if (err) {
                    console.error("Unable to send message. Error JSON:", JSON.stringify(err, null, 2));
                } else {
                    console.log("Results from sending message: ", JSON.stringify(data, null, 2));
                }
            });
        }
    });
    callback(null, `Successfully processed ${event.Records.length} records.`);
};   
```

### read consistency

When your application writes data to a DynamoDB table and receives an HTTP 200 response (OK), that means the write completed successfully and has been durably persisted. DynamoDB provides __read-committed isolation__ and ensures that read operations always return committed values for an item. The read will never present a view to the item from a write which did not ultimately succeed. Read-committed isolation does not prevent modifications of the item immediately after the read operation.

__Eventually consistent is the default read consistent model__ for all read operations. When issuing eventually consistent reads to a DynamoDB table or an index, the responses may not reflect the results of a recently completed write operation. If you repeat your read request after a short time, the response should eventually return the more recent item.

Read operations such as GetItem, Query, and Scan provide an optional __ConsistentRead parameter__. If you set ConsistentRead to true, DynamoDB returns a response with the most up-to-date data, reflecting the updates from all prior write operations that were successful. Strongly consistent reads are only supported on tables and local secondary indexes.

### global tables

Specific benefits for using global tables include:
- Replicating your DynamoDB tables automatically across your choice of AWS Regions
- Eliminating the difficult work of replicating data between Regions and resolving update conflicts, so you can focus on your application's business logic.
- Helping your applications stay highly available even in the unlikely event of isolation or degradation of an entire Region.

Transactional operations provide atomicity, consistency, isolation, and durability (ACID) guarantees only within the region where the write is made originally. Transactions are not supported across regions in global tables. 

There is no performance impact on source regions when adding new replicas.

Each global table produces an independent stream based on all its writes, regardless of the origination point for those writes. You can choose to consume this DynamoDB stream in one Region or in all Regions independently.

If you want processed local writes but not replicated writes, you can add your own Region attribute to each item. Then you can use a Lambda event filter to invoke only the Lambda for writes in the local Region.

Any changes made to any item in any replica table are replicated to all the other replicas within the same global table. In a global table, a newly written item is usually propagated to all replica tables within a second.

With a global table, each replica table stores the same set of data items. DynamoDB does not support partial replication of only some of the items.

DynamoDB __doesn't support strongly consistent reads across Regions.__ 
Therefore, if you write to one Region and read from another Region, the read response might include stale data that doesn't reflect the results of recently completed writes in the other Region. 

If applications update the same item in different Regions at about the same time, conflicts can arise. 
To help ensure eventual consistency, DynamoDB global tables use a __last writer wins__ reconciliation between concurrent updates, in which DynamoDB makes a best effort to determine the last writer.

### Transactions

You can use the DynamoDB transactional read and write APIs to manage complex business workflows that require adding, updating, or deleting multiple items as a single, all-or-nothing operation.

With the transaction write API, you can group multiple __Put, Update, Delete, and ConditionCheck actions__. You can then submit the actions as a single __TransactWriteItems__ operation that either succeeds or fails as a unit. The same is true for __multiple Get actions__, which you can group and submit as a single __TransactGetItems__ operation.

The aggregate size of the items in the transaction cannot exceed 4 MB

You can't target the same item with multiple operations within the same transaction

Idempotent: If the original TransactWriteItems call was successful, then subsequent TransactWriteItems calls with the same client token return successfully without making any changes.

The isolation levels of transactional operations (TransactWriteItems or TransactGetItems) and other operations are as follows:
- SERIALIZABLE:  results of multiple concurrent operations are the same as if no operation begins until the previous one has finished.
- READ-COMMITTED: ensures that read operations always return committed values for an item - the read will never present a view to the item representing a state from a transactional write which did not ultimately succeed

Transaction __conflicts__ can occur in the following scenarios:
- A PutItem, UpdateItem, or DeleteItem request for an item conflicts with an ongoing TransactWriteItems request that includes the same item.
- An item within a TransactWriteItems request is part of another ongoing TransactWriteItems request.
- An item within a TransactGetItems request is part of an ongoing TransactWriteItems, BatchWriteItem, PutItem, UpdateItem, or DeleteItem request.

__TransactWriteItems and TransactGetItems are both supported in DynamoDB Accelerator (DAX)__ with the same isolation levels as in DynamoDB.

```
...
final String ORDER_PARTITION_KEY = "OrderId";
final String ORDER_TABLE_NAME = "Orders";

HashMap<String, AttributeValue> orderItem = new HashMap<>();
orderItem.put(ORDER_PARTITION_KEY, new AttributeValue(orderId));
orderItem.put(PRODUCT_PARTITION_KEY, new AttributeValue(productKey));
orderItem.put(CUSTOMER_PARTITION_KEY, new AttributeValue(customerId));
orderItem.put("OrderStatus", new AttributeValue("CONFIRMED"));
orderItem.put("OrderTotal", new AttributeValue("100"));

Put createOrder = new Put()
    .withTableName(ORDER_TABLE_NAME)
    .withItem(orderItem)
    .withReturnValuesOnConditionCheckFailure(ReturnValuesOnConditionCheckFailure.ALL_OLD)
    .withConditionExpression("attribute_not_exists(" + ORDER_PARTITION_KEY + ")");
    Collection<TransactWriteItem> actions = Arrays.asList(
        new TransactWriteItem().withConditionCheck(checkCustomerValid),
        new TransactWriteItem().withUpdate(markItemSold),
        new TransactWriteItem().withPut(createOrder));

...
TransactWriteItemsRequest placeOrderTransaction = new TransactWriteItemsRequest()
    .withTransactItems(actions)
    .withReturnConsumedCapacity(ReturnConsumedCapacity.TOTAL);

// Run the transaction and process the result.
try {
    client.transactWriteItems(placeOrderTransaction);
    System.out.println("Transaction Successful");

} catch (ResourceNotFoundException rnf) {
    System.err.println("One of the table involved in the transaction is not found" + rnf.getMessage());
} catch (InternalServerErrorException ise) {
    System.err.println("Internal Server Error" + ise.getMessage());
} catch (TransactionCanceledException tce) {
    System.out.println("Transaction Canceled " + tce.getMessage());
}
```

### In-memory acceleration with DynamoDB Accelerator

DAX is a DynamoDB-compatible caching service that enables you to benefit from fast __in-memory__ performance for demanding applications.

As an in-memory cache, DAX reduces the response times of __eventually consistent read workloads__ by an __order of magnitude__ from single-digit milliseconds to microseconds

### Data modeling

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/data-modeling.html

__Item collection__ is a group of items that share the same partition key value, which means the items are related. Item collections are the primary mechanism to model one-to-many relationships in DynamoDB

__Single table design__ is a pattern that allows you to store multiple types (entities) of data in a single DynamoDB table. It aims to optimize data access patterns, improve performance, and reduce costs by eliminating the need for maintaining multiple tables and complex relationships between them.

__Multiple table design__ is a pattern that is more like a traditional database design where you store a single type(entity) of data in a each DynamoDB table. Data within each table will still be organized by partition key so performance within a single entity type will be optimized for scalability and performance, but queries across multiple tables must be done independently.

#### Composite sort key building

One of the most critical patterns we can use to develop a logical hierarchy of our data in DynamoDB is a __composite sort key__. 
The most common style for designing one is with each layer of the hierarchy (parent layer > child layer > grandchild layer) separated by a hashtag. For example, PARENT#CHILD#GRANDCHILD#ETC.

While a partition key in DynamoDB always requires the exact value to query for data, we can apply a partial condition to the sort key from left to right similar to traversing a binary tree.

| Primary Key                         |
| Partition key: PK   | Sork key: SK  |
|---------------------|-------------- |
| UserID              | CART#ACTIVE#Apples
|                     | CART#ACTIVE#Bananas

#### Multi-tenancy building block

We want to design the schema in a way that keeps all data from a single tenant in its own logical partition of the table. This leverages the concept of an Item Collection, which is a term for all items in a DynamoDB table with the same partition key. 

| Primary Key                         |
| Partition key: PK   | Sork key: SK  | Attributes
|---------------------|---------------| ------------
| UserOne             | PhotoID1      | ImageURL: https://.../file1.jpg
| UserOne             | PhotoID2      | ImageURL: https://.../file2.jpg
| UserTwo             | PhotoID3      | ImageURL: https://.../file3.jpg

#### Time to live building block

To facilitate data aging out from DynamoDB, it has a feature called time to live (TTL). The TTL feature allows you to define a specific attribute at the table level that needs monitoring for items with an epoch timestamp (that's in the past). This allows you to delete expired records from the table for free.

| Primary Key                               |
| Partition key: PK   | Sork key: SK        | Attributes
|---------------------|---------------------| ------------
| UserID              | 2030-06-10T11:45:01 | TTL: 1982736353
|                     |                     | Message: "Hello"

In this example, we have an application designed to let a user create messages that are short-lived. When a message is created in DynamoDB, the TTL attribute is set to a date seven days in the future by the application code. In roughly seven days, DynamoDB will see that the epoch timestamp of these items is in the past and delete them.

Since the deletes done by TTL are free, it is strongly recommended to use this feature to remove historical data from the table

#### Vertical partitioning building block

Storing all related data within a single JSON document. While DynamoDB supports JSON data types, it does not support excuting KeyConditions on nested JSON. Since KeyConditions are what dictate how much data is read from disk and effectively how many RCUs a query consumes, this can result in inefficiencies at scale. To better optimize the writes and reads of DynamoDB, we recommend breaking apart the document's individual entities into individual DynamoDB items, also referred to as vertical partitioning.

| Primary Key                                      |
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| UserOne             | PhoAddress#USA#CA#LA#90029 | StreetAddress: "1 main st"
|                     | CART#ACTIVE#Coffee         | CoffeeSKU: "yuioy21yui3y21"

#### Examples
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/data-modeling-schemas.html

##### Social network access patterns

Data access patterns:
- getUserInfoByUserID
- getFollowerListByUserID
- getFollowingListByUserID
- getPostListByUserID
- getUserLikesByPostID
- getLikeCountByPostID
- getTimelineByUserID

__Pattern: getUserInfoByUserID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345             | "count"                    | NumberOfFollowers: 12345; NumberOfFollowing: 10; NumberOfPosts: 1872
|                     | "info"                     | Name: KJ; imageURL: http://; Country: "USA"

__Pattern: getFollowerListByUserID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#follower    | u#1111                     | 
| u#12345#follower    | u#2222222                  | 

__Pattern: getFollowingListByUserID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#following   | u#44444444                 | 
| u#12345#following   | u#55555555                 | 

__Pattern: getPostListByUserID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#fpost       | p#12345                    | Content: "post content"; imageUrl: "http://..."; timestamp: 12355

__Pattern: getUserLikesByPostID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#likelist    | u#44444444                 | 
| u#12345#likelist    | u#55555555                 | 

__Pattern: getLikeCountByPostID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#likecount   | count                      | count=100

__Pattern: getTimelineByUserID:__
| Partition key: PK   | Sork key: SK               | Attributes
|---------------------|----------------------------| ------------
| u#12345#timeline    | p#123#u5678                | timestamp: 12345

Every time a user writes a post, their follower list is read and their userID and postID are slowly entered into the timeline key of all its followers. Then, when your application starts, you can read the timeline key with the Query operation and fill the timeline screen with a combination of userID and postID using the BatchGetItem operation for any new items. You cannot read the timeline with an API call, but this is a more cost effective solution if the posts could be edited frequently.


# Redshift <a id="Redshift"></a>

# AWS Database interview questions <a id="InterviewQuestions"></a>

### If I launch a standby RDS instance, will it be in the same Availability Zone as my primary?
No

the purpose of having a standby instance is to avoid an infrastructure failure (if it happens), therefore the standby instance is stored in a different availability zone, which is a physically different independent infrastructure.

### When would I prefer Provisioned IOPS over Standard RDS storage?
If you have batch-oriented workloads

Provisioned IOPS deliver high IO rates but on the other hand it is expensive as well. 
Batch processing workloads do not require manual intervention they enable full utilization of systems, therefore a provisioned IOPS will be preferred for batch oriented workload.

### How is Amazon RDS, DynamoDB and Redshift different?

Amazon RDS is a database management service for relational databases, it manages patching, upgrading, backing up of data etc. of databases for you without your intervention. RDS is a Db management service for structured data only.
DynamoDB, on the other hand, is a NoSQL database service, NoSQL deals with unstructured data.
Redshift, is an entirely different service, it is a data warehouse product and is used in data analysis.

### If I am running my DB Instance as a Multi-AZ deployment, can I use the standby DB Instance for read or write operations along with primary DB instance?

No

Standby DB instance cannot be used with primary DB instance in parallel, as the former is solely used for standby purposes, it cannot be used unless the primary instance goes down

### Explain the concept of Multi-AZ deployments in Amazon RDS

Multi-AZ (Availability Zone) deployments involve maintaining a standby replica of the primary database in a different Availability Zone for high availability and automatic failover.

### How can you scale the compute and storage resources of an Amazon RDS instance?

You can scale resources vertically by modifying the instance type or horizontally by adding read replicas.


