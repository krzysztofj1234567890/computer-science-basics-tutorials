# Databases

## Relational database management systems (rdbms)

### Relational algebra

E.F. Codd proposed the relational Model to model data in the form of relations or tables.

#### What is the Relational Model?

The relational model represents how data is stored in Relational Databases. 
Think of a relation as a file that is formatted in a special way.
Relations are Two-Dimensional Table.
A relational database consists of a collection of tables, each of which is assigned a unique name.

Table: STUDENT

| id  | name  | phone      | age  |
|---- | ----- | ---------- | ---- |
| 1   | Chris | 1234567890 | 40   |
| 2   | Tom   | 1234567891 | 10   |
| 3   | Kate  | 1234567892 | 11   |
| 4   | Mom   | 1234567893 | NULL |

* Attribute: Attributes are the properties that define an entity. e.g.: NAME, ADDRESS

* Column: The column represents the set of values for a particular attribute. The column 'id' = [1,2,3,4]

* Tuple: Each row in the relation is known as a tuple. The above relation contains 4 tuples

* NULL Values: The value which is not known or unavailable is called a NULL value.

[More about relational model](https://www.geeksforgeeks.org/relational-model-in-dbms/?ref=lbp)

[Read about Codd](https://www.ibm.com/ibm/history/ibm100/us/en/icons/reldb/)

[Relational algebra](https://en.wikipedia.org/wiki/Relational_algebra)

Relational algebra mainly provides a theoretical foundation for relational databases and SQL. 
The main purpose of using Relational Algebra is to define operators that transform one or more input relations into an output relation.

Operators on relations:
* selection
* projection
* union
* etc.

[Introduction of Relational Algebra in DBMS](https://www.geeksforgeeks.org/introduction-of-relational-algebra-in-dbms/?ref=lbp)

[Anomalies in Relational Model](https://www.geeksforgeeks.org/anomalies-in-relational-model/?ref=lbp)

### Keys

Keys are one of the basic requirements of a relational database model. 
It is widely used to identify the tuples(rows) uniquely in the table. 

[Keys in Relational Model](https://www.geeksforgeeks.org/types-of-keys-in-relational-model-candidate-super-primary-alternate-and-foreign/?ref=lbp)

### Extended operations

Table: STUDENT

| id  | name  | phone      | age  |
|---- | ----- | ---------- | ---- |
| 1   | Chris | 1234567890 | 40   |
| 2   | Tom   | 1234567891 | 10   |
| 3   | Kate  | 1234567892 | 11   |
| 4   | Mom   | 1234567893 | NULL |

Table: ADDRESS

| id  | street     | city     | zip   | 
|---- | ---------- | -------- | ------| 
| 1   | 1 main     | Boston   | 87654 |
| 2   | 1 broadway | New York | 01234 |

Table: STUDENT_ADDRESS

| student_id | address_id |
| ---------- | ---------- |
| 1          | 1          |
| 2          | 1          |
| 3          | 2          |
| 1          | 1          |

[Extended Operators in Relational Algebra](https://www.geeksforgeeks.org/extended-operators-in-relational-algebra/?ref=lbp)

### RDBMS design

#### Normalization

Database normalization is the process of organizing the attributes of the database to reduce or eliminate data redundancy (having the same data but at different places)

[Introduction of Database Normalization](https://www.geeksforgeeks.org/introduction-of-database-normalization/?ref=lbp)

[Normal Forms in DBMS](https://www.geeksforgeeks.org/normal-forms-in-dbms/?ref=lbp)

### SQL

SQL is a standard database language used to access and manipulate data in databases. SQL stands for Structured Query Language.

#### DDL = data definition language

```
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    phone VARCHAR(10),
    age int(2)
);
```

#### DML = data manipulation language

INSERT INTO student VALUES ('1','Chris','1234567890', 40);

#### DQL = data query language

SELECT name, age FROM student


[SQL tutorial](https://www.geeksforgeeks.org/sql-ddl-dql-dml-dcl-tcl-commands/?ref=lbp)

#### Joins

[SQL joins](https://www.geeksforgeeks.org/sql-join-set-1-inner-left-right-and-full-joins/?ref=lbp)

#### Group by

[SQL group by](https://www.geeksforgeeks.org/sql-group-by/?ref=lbp)

### Run PostgreSQL

#### run container on your laptop

#### SQL interface

### Transactions and concurrency

Transaction is a set of logically related operations is known as a transaction

[rdbms concurrency control](https://www.geeksforgeeks.org/concurrency-control-in-dbms/?ref=lbp)

#### ACID

* Atomicity defines all the elements that make up a complete database transaction.
* Consistency defines the rules for maintaining data points in a correct state after a transaction.
* Isolation keeps the effect of a transaction invisible to others until it is committed, to avoid confusion.
* Durability ensures that data changes become permanent once the transaction is committed.

[ACID](https://www.geeksforgeeks.org/acid-properties-in-dbms/?ref=lbp)

## Install and run postgres database

We will install:
* postgres rdbms: relational database system
* pgadmin : web based administration tool for the PostgreSQL database

### Install postgres and pgadmin using containers

I assume that you have docker desktop installed.

#### start postgres container:

You will install the following image: https://hub.docker.com/_/postgres

To do that open docker desktop -> images and search for postgres. Find the above docker image, download it and run it.

Start the container with the following settings:
* Name:	postgres
* Port: 	5432:5432
* POSTGRES_PASSWORD: create and remember the password

#### start pgadmin container:

You will install the following image: https://hub.docker.com/r/dpage/pgadmin4/

To do that open docker desktop -> images and search for pgadmin. Find the above docker image, download it and run it.

Start the container with the following settings:
* Name:	 pgadmin
* HOST PORT:	1234:80
* PGADMIN_DEFAULT_EMAIL: your fake email
* PGADMIN_DEFAULT_PASSWORD: create and remember the password

#### connect containers

Both containers work in isolation and do not see each other. They need to be put in the same network. To do that you will use 'docker network' command: create new network and add both containers to it:

In cmd (command line) type the following:

```
docker network ls
docker network create --driver bridge pgnetwork
docker network connect pgnetwork pgadmin
docker network connect pgnetwork postgres
docker network inspect pgnetwork
```

#### connect to database using pgadmin

Now you will use pgadmin web client to connect and manage your postgres database.

1. In your web browser open the following url: http://localhost:1234/browser/. Note that the '1234' is the port you have exposed in the setting of the docker container configuration
2. Login using the credentials specified in your docker container settings
3. Add Server
    1. General
        1. Name: postgres
    2. Connection
        1. Hostname: postgres
	    2. Port: 5432
	    3. database: postgres
	    4. username: postgres
	    5. password: password you have defined

You are connected to the database server

#### create database

Now create your database. In the pgadmin session do the following:

Object Explorer -> Query Tool

```
CREATE DATABASE TomDB ;
```

Refresh your panel to see the created database:

Object Explorer -> Servers -> postgres -> Refresh (right click)

#### Connect to the new database

Right click on 'tomdb' database. Select the connection: 'tomdb/postgres@postgres'

#### create table

Object Explorer -> Query Tool

```
CREATE TABLE student (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    phone VARCHAR(10),
    age int
);
```

Execute query

Insert new rows:

```
INSERT INTO student VALUES ('1','Chris','1234567890', 40);
INSERT INTO student VALUES ('2','Tom','1234567891', 10);
INSERT INTO student VALUES ('3','Kate','1234567892', 11);
INSERT INTO student VALUES ('4','Mom','1234567893', NULL);
```

Execute query

#### Run some queries

Object Explorer -> Query Tool

```
SELECT * FROM student
```

Execute query

