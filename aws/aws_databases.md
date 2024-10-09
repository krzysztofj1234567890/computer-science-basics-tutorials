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
- Provide access to your DB instance in your VPC by creating a security group

## RDS for PostgreSQL


