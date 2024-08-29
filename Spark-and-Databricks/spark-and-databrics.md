# Spark - the definitive Guide by Bill Chambers and Matei Zaharia 2018

Apache Spark is a unified computing engine and a set of libraries for parallel data processing on computer clusters

## A Gentle Introduction to Spark

Spark Applications consist of a driver process and a set of executor processes. 

The __driver process__ runs your main() function, sits on a node in the cluster, and is responsible for three things:
maintaining information about the Spark Application; responding to a user’s program or input;
and analyzing, distributing, and scheduling work across the executors.

The __executors__ are responsible for actually carrying out the work that the driver assigns them.
This means that each executor is responsible for only two things: executing code assigned to it
by the driver, and reporting the state of the computation on that executor back to the driver node.

Spark employs a __cluster manager__ that keeps track of the resources available

![ Spark Architecture ](./images/spark_architecture.png)

### The SparkSession

You control your Spark Application through a driver process called the SparkSession. The SparkSession instance is the way Spark executes
user-defined manipulations across the cluster. There is a one-to-one correspondence between a
SparkSession and a Spark Application. In Scala and Python, the variable is available as spark
when you start the console. 

### DataFrame

A DataFrame is the most common Structured API and simply represents a table of data with
rows and columns. The list that defines the columns and the types within those columns is called
the schema.

Spark DataFrame can span thousands of computers.

To allow every executor to perform work in parallel, Spark breaks up the data into chunks called
__partitions__. A partition is a collection of rows that sit on one physical machine in your cluster.

### DataSets

A type-safe version of Spark’s structured API called Datasets, for writing statically typed code in Java and Scala. The Dataset API is not available in Python and R, because those languages are dynamically typed.

The Dataset API gives users the ability to assign a Java/Scala class to the records within a DataFrame and manipulate it as a
collection of typed objects, similar to a Java ArrayList or Scala Seq.

The APIs available on Datasets are type-safe, meaning that you cannot accidentally view the objects in a Dataset as
being of another class than the class you put in initially. This makes Datasets especially attractive
for writing large applications, with which multiple software engineers must interact through
well-defined interfaces.

One final advantage is that when you call collect or take on a Dataset, it will collect objects of
the proper type in your Dataset, not DataFrame Rows.

```
// in Scala
case class Flight(DEST_COUNTRY_NAME: String,
    RIGIN_COUNTRY_NAME: String,
    count: BigInt)
val flightsDF = spark.read.parquet("/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight]

flights
.filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
.map(flight_row => flight_row)
.take(5)
```

### Transformations

In Spark, the core data structures are immutable, meaning they cannot be changed after they’re created.

To “change” a DataFrame, you need to instruct Spark how you would like to modify it to do what you want. These instructions are called transformations.

```
# in Python
divisBy2 = myRange.where("number % 2 = 0")
```

Notice that these return no output. This is because we specified only an abstract transformation, and Spark will not act on transformations until we call an action.

Lazy evaulation means that Spark will wait until the very last moment to execute the graph of
computation instructions. In Spark, instead of modifying the data immediately when you express
some operation, you build up a plan of transformations that you would like to apply to your
source data. By waiting until the last minute to execute the code, Spark compiles this plan from
your raw DataFrame transformations to a streamlined physical plan that will run as efficiently as
possible across the cluster. This provides immense benefits because Spark can optimize the
entire data flow from end to end. 

### Actions

Transformations allow us to build up our logical transformation plan. To trigger the computation,
we run an action. An action instructs Spark to compute a result from a series of transformations.

The simplest action is count, which gives us the total number of records in the DataFrame:
```
divisBy2.count()
```

### Example

Data:
```
head /data/flight-data/csv/2015-summary.csv
DEST_COUNTRY_NAME,ORIGIN_COUNTRY_NAME,count
United States,Romania,15
United States,Croatia,1
United States,Ireland,344
```

Code:

```
# in Python
flightData2015 = spark\
.read\
.option("inferSchema", "true")\
.option("header", "true")\
.csv("/data/flight-data/csv/2015-summary.csv"

flightData2015.take(3)

flightData2015.sort("count").explain()

```

By default, when we perform a shuffle, Spark outputs 200 shuffle partitions. Let’s set this value to 5 to reduce the number of the output
partitions from the shuffle:

```
spark.conf.set("spark.sql.shuffle.partitions", "5")
flightData2015.sort("count").take(2)
```

### Spark SQL

With Spark SQL, you can register any DataFrame as a table or view (a temporary table) and query it using pure SQL

```
flightData2015.createOrReplaceTempView("flight_data_2015")

# in Python
sqlWay = spark.sql("""
SELECT DEST_COUNTRY_NAME, count(1)
FROM flight_data_2015
GROUP BY DEST_COUNTRY_NAME
""")

dataFrameWay = flightData2015\
.groupBy("DEST_COUNTRY_NAME")\
.count()

sqlWay.explain()
dataFrameWay.explain()
```

### Structured Streaming

With Structured Streaming, you can take the same operations that you perform in
batch mode using Spark’s structured APIs and run them in a streaming fashion. This can reduce
latency and allow for incremental processing. The best thing about Structured Streaming is that it
allows you to rapidly and quickly extract value out of streaming systems with virtually no code
changes. 

Sample of the data:
```
InvoiceNo,StockCode,Description,Quantity,InvoiceDate,UnitPrice,CustomerID,Country
536365,85123A,WHITE HANGING HEART T-LIGHT HOLDER,6,2010-12-01 08:26:00,2.55,17...
536365,71053,WHITE METAL LANTERN,6,2010-12-01 08:26:00,3.39,17850.0,United Kin...
536365,84406B,CREAM CUPID HEARTS COAT HANGER,8,2010-12-01 08:26:00,2.75,17850
```

We’ll also create a schema from this static dataset:
```
# in Python
staticDataFrame = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/by-day/*.csv")

staticDataFrame.createOrReplaceTempView("retail_data")
staticSchema = staticDataFrame.schema
```

The window function will include all data from each day in the aggregation.

The static DataFrame version:
```
# in Python
from pyspark.sql.functions import window, column, desc, col

staticDataFrame\
    .selectExpr(
        "CustomerId",
        "(UnitPrice * Quantity) as total_cost",
        "InvoiceDate")\
    .groupBy(
        col("CustomerId"), window(col("InvoiceDate"), "1 day"))\
    .sum("total_cost")\
    .show(5)
```

Streaming code. The biggest change is that we used readStream instead of read, additionally you’ll notice the maxFilesPerTrigger option, which simply specifies the number of files we should read in at once

```
# in Python
streamingDataFrame = spark.readStream\
    .schema(staticSchema)\
    .option("maxFilesPerTrigger", 1)\
    .format("csv")\
    .option("header", "true")\
    .load("/data/retail-data/by-day/*.csv")

purchaseByCustomerPerHour = streamingDataFrame\
    .selectExpr(
"       CustomerId",
        "(UnitPrice * Quantity) as total_cost",
        "InvoiceDate")\
    .groupBy(
        col("CustomerId"), window(col("InvoiceDate"), "1 day"))\
    .sum("total_cost")
```

## Structured API Overview

### Structured API Execution

steps:
1. Write DataFrame/Dataset/SQL Code.
2. If valid code, Spark converts this to a Logical Plan.
3. Spark transforms this Logical Plan to a Physical Plan, checking for optimizations along the way.
4. Spark then executes this Physical Plan (RDD manipulations) on the cluster.

__Logical Planning__ represents a set of abstract transformations that do not refer to executors or drivers. 
This plan is unresolved because although your code might be valid, the tables or columns that it refers to might or might
not exist. Spark uses the catalog, a repository of all table and DataFrame information, to resolve
columns and tables in the analyzer. The analyzer might reject the unresolved logical plan if the
required table or column name does not exist in the catalog. If the analyzer can resolve it, the
result is passed through the Catalyst Optimizer, a collection of rules that attempt to optimize the
logical plan by pushing down predicates or selections.

After successfully creating an optimized logical plan, Spark then begins the __physical planning__
process. The physical plan, often called a Spark plan, specifies how the logical plan will execute
on the cluster by generating different physical execution strategies and comparing them through a cost model.

An example of the cost comparison might be choosing how to perform a given join by looking at the physical attributes of a given table

Physical planning results in a series of RDDs and transformations

Upon selecting a physical plan, Spark runs all of this code over RDDs, the lower-level programming interface of Spark

## Basic Structured Operations

A __schema__ defines the column names and types of a DataFrame.

```
# in Python
spark.read.format("json").load("/data/flight-data/json/2015-summary.json").schema
```

To Spark, __columns__ are logical constructions that simply represent a value computed on a per-record basis by means of an expression. This means that to have a real value for a column, we need to have a row; and to have a row, we need to have a DataFrame. You cannot manipulate an
individual column outside the context of a DataFrame; you must use Spark transformations
within a DataFrame to modify the contents of a column.

An __expression__ is a set of transformations on one or more values in a record in a DataFrame.

Columns provide a subset of expression functionality. 

In Spark, each __row__ in a DataFrame is a single record. Spark represents this record as an object of
type Row. Spark manipulates Row objects using column expressions in order to produce usable
values. Row objects internally represent arrays of bytes. 

### DataFrame Transformations

To create dataFrame we can:
* create DataFrames from raw data sources
```
# in Python
df = spark.read.format("json").load("/data/flight-data/json/2015-summary.json")
df.createOrReplaceTempView("dfTable"
```

* taking a set of rows and converting them to a DataFrame
```
# in Python
from pyspark.sql import Row
from pyspark.sql.types import StructField, StructType, StringType, LongType
myManualSchema = StructType([
    StructField("some", StringType(), True),
    StructField("col", StringType(), True),
    StructField("names", LongType(), False)
])
myRow = Row("Hello", None, 1)
myDf = spark.createDataFrame([myRow], myManualSchema)
myDf.show()
```

DataFrame transformations:
* We can add rows or columns
* We can remove rows or columns
* We can transform a row into a column (or vice versa)
* We can change the order of rows based on the values in columns

```
# in Python
df.withColumn("numberOne", lit(1)).show(2)
```

# Examples

## Notebook

## DataBrics job on azure

