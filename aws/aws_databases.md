# RDS

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

### RDS Proxy

You can set up the connection between your Lambda function and your DB instance through RDS Proxy to improve your database performance and resiliency. Often, Lambda functions make frequent, short database connections that benefit from connection pooling that RDS Proxy offers.

## RDS for PostgreSQL

### Creating and connecting to a PostgreSQL DB instance

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


## Aurora

Amazon Aurora (Aurora) is a fully managed relational database engine that's compatible with MySQL and PostgreSQL.

The speed and reliability of high-end commercial databases with the simplicity and cost-effectiveness of open-source databases. 
The code, tools, and applications you use today with your existing MySQL and PostgreSQL databases can be used with Aurora. 
With some workloads, Aurora can deliver up to five times the throughput of MySQL and up to three times the throughput of PostgreSQL without requiring changes

Aurora includes a high-performance storage subsystem. The underlying storage grows automatically as needed.

### Aurora DB clusters

An Amazon Aurora DB cluster consists of one or more DB instances and a cluster volume that manages the data for those DB instances. 
An Aurora cluster volume is a virtual database storage volume that spans multiple Availability Zones, with each Availability Zone having a copy of the DB cluster data. 

Two types of DB instances make up an Aurora DB cluster:
- Primary (writer) DB instance – Supports read and write operations, and performs all of the data modifications to the cluster volume. Each Aurora DB cluster has one primary DB instance.
- Aurora Replica (reader DB instance) – Connects to the same storage volume as the primary DB instance but supports only read operations. Each Aurora DB cluster can have up to 15 Aurora Replicas

### Aurora Features

- blue-green deployments: By using Amazon RDS Blue/Green Deployments, you can make changes to the database in the staging environment without affecting the production environment. 
- export data to S3: You can export Aurora DB cluster data to an Amazon S3 bucket. After the data is exported, you can analyze the exported data directly through tools like Amazon Athena or Amazon Redshift Spectrum
- global databases: An Aurora global database is a single database that spans multiple AWS Regions, enabling low-latency global reads and disaster recovery from any Region-wide outage
- zero-ETL: Amazon Aurora zero-ETL integrations with Amazon Redshift is a fully managed solution for making transactional data available in Amazon Redshift after it's written to an Aurora cluster.
- RDS proxy: Amazon RDS Proxy is a fully managed, highly available database proxy that makes applications more scalable by pooling and sharing established database connections.
- Serverless: Aurora Serverless v2 is an on-demand, auto-scaling feature designed to be a cost-effective approach to running intermittent or unpredictable workloads on Amazon Aurora. It automatically scales capacity up or down as needed by your applications.

### Aurora global databases

An Aurora global database consists of one primary AWS Region where your data is written, and up to five read-only secondary AWS Regions. 
You issue write operations directly to the primary DB cluster in the primary AWS Region. 
Aurora replicates data to the secondary AWS Regions using dedicated infrastructure, with latency typically under a second.

### RDS Proxy

- By using Amazon RDS Proxy, you can allow your applications to __pool and share database connections__ to improve their ability to __scale__. 
- RDS Proxy makes applications more __resilient__ to database failures by __automatically connecting to a standby DB instance while preserving application connections__. 
- By using RDS Proxy, you can also enforce __AWS Identity and Access Management__ (IAM) __authentication__ for databases, and securely store credentials in AWS Secrets Manager. 

RDS Proxy __queues__ or __throttles__ application connections that can't be served immediately from the connection pool. 
Although latencies might increase, your application can continue to scale without abruptly failing or overwhelming the database

### Aurora zero-ETL integrations with Amazon Redshift

Enables near real-time analytics and machine learning (ML) using Amazon Redshift on petabytes of transactional data from Aurora

The source DB cluster must be in the same Region as the target Amazon Redshift data warehouse.

### Aurora Serverless v2

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

# DynamoDB

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
It offers two streaming models for CDC: DynamoDB Streams and Kinesis Data Streams for DynamoDB
- __Secondary indexes__: DynamoDB offers the option to create both global and local secondary indexes, which let you query the table data using an alternate key.
- __S3 integration__: Integrating DynamoDB with Amazon S3 enables you to easily export data to an Amazon S3 bucket for analytics and machine learning
- __Zero-ETL integration__: DynamoDB supports zero-ETL integration with Amazon Redshift and Amazon OpenSearch Service. 
These integrations enable you to run complex analytics and use advanced search capabilities on your DynamoDB table data.
- __DAX (Caching)__: is a fully managed, highly available caching service built for DynamoDB. DAX delivers up to 10 times performance improvement – from milliseconds to microseconds – even at millions of requests per second. DAX does all the heavy lifting required to add in-memory acceleration to your DynamoDB tables, without requiring you to manage cache invalidation, data population, or cluster management.
- Resilience: By default, DynamoDB automatically replicates your data across three Availability Zones to provide high durability and a 99.99% availability SLA.

Continue: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/getting-started-step-1.html


# Redshift

# AWS Database interview questions

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


