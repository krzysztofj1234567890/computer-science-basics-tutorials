# Data Lakehouse Architecture

## What is a Data Lake vs Lakehouse?

__Data Lake:__
- __Centralized__ storage for raw + processed data
- __Cheap__, scalable object storage
- __Schema-on-read__
- Historically weak on governance & performance

__Lakehouse__: A data lake + warehouse features:
- __ACID__ transactions
- __Schema__ enforcement & evolution
- __Time travel__
- Performance optimizations
- SQL-first analytics
- Key enablers: Delta Lake, Apache Iceberg, Apache Hudi

## Ingestion Layer

### Batch ingestion

- Used for:
  - Databases
  - Files
  - Periodic exports
- Techniques
  - Full loads (initial)
  - Incremental loads (timestamp, watermark)
  - CDC (change data capture)
- Tools
  - Airflow, Dagster (orchestration)
  - Spark, Flink (processing)
  - Debezium, Fivetran (CDC)

### Streaming ingestion

- Used for:
  - Events
  - Logs
  - Near-real-time analytics
- Flow: Source → Kafka/Kinesis → Stream processor → Lake
- Design choices
  - Exactly-once vs at-least-once
  - Event time vs processing time
  - Late-arriving data handling

## Storage Layer

__Object storage__: Foundation of the lake/lakehouse:
- __S3 / ADLS / GCS__
- Cheap, durable, infinite scale
- Append-only by nature

__File formats__:
- Columnar:
- __Parquet__
    - Predicate pushdown
    - Compression
    - Vectorized reads
- Table formats (sit on top of parquet)
- __Delta Lake__: Simplicity, ecosystem
- __Apache Iceberg__: Open spec, engine-agnostic
- __Apache Hudi__: Streaming & upserts
- table formats add:
    - ACID transactions
    - Schema enforcement/evolution
    - Snapshots & time travel
    - Compaction & file management
  
__Partitioning & layout__:
- Good partitions
  - High cardinality filters (date, region)
  - Avoid over-partitioning
- Optimizations
  - File compaction
  - Z-ordering / clustering
  - Data skipping indexes

## Compute Engines

Compute is decoupled from storage.

Multiple engines, one source of truth.

### Batch & SQL engines

| Engine         | Strength                   |
| -------------- | -------------------------- |
| Spark          | Heavy ETL, ML              |
| Trino / Presto | Fast interactive SQL       |
| DuckDB         | Local / embedded analytics |
| Snowflake      | Managed, proprietary       |


### Streaming engines

- Spark Structured Streaming
- Flink
- Kafka Streams

## Metadata, Catalog & Lineage

Without strong metadata:
- Data is untrustworthy
- Self-service fails
- Governance is manual

Metadata types:
- Technical metadata
  - Schemas
  - Partitions
  - File locations
  - Statistics
- Business metadata
  - Definitions
  - Owners
  - SLAs
  - Sensitivity

| Tool               | Purpose              |
| ------------------ | -------------------- |
| Hive Metastore     | Legacy               |
| AWS Glue           | Managed              |
| Unity Catalog      | Databricks           |
| Apache Polaris     | Open Iceberg catalog |
| DataHub / Amundsen | Discovery & lineage  |

## Governance & Security

### Access control:
- Table / column / row-level security
- Attribute-based access control (ABAC)

Implemented via
- Catalog
- Query engine
- Identity provider

### Data classification:
- PII
- PHI
- Financial data

Drives:
- Masking
- Tokenization
- Audit logging

### Lineage

Tracks: Source → transformations → consumption

Critical for:
- Impact analysis
- Compliance
- Debugging

### Data quality

Implemented as:
- Validation rules
- Freshness checks
- Volume anomaly detection

Often integrated into:
- Pipelines
- Orchestration layer

## Database / User Experience

SQL experience: A good lakehouse feels like a database:
- Fast queries
- Predictable performance
- Stable schemas
- Transactions

Achieved via
- Materialized views
- Aggregations
- Caching
- Semantic layers

### Semantic layer: Maps technical data → business concepts.

Examples:
- dbt metrics
- LookML
- Cube

Benefits:
- Consistent definitions
- Less duplicated logic
- Easier self-service

### BI & ML consumption

BI tools: 
- Tableau, Power BI, Looker, 
- Direct query or extracts

ML workflows
- Feature stores

## Modern Lakehouse Reference Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                         │
│                                                              │
│  OLTP DBs   SaaS Apps   Logs   Events   Files   IoT   APIs    │
└───────────────┬───────────────┬───────────────┬─────────────┘
                │               │               │
                ▼               ▼               ▼
┌──────────────────────────────────────────────────────────────┐
│                      INGESTION LAYER                          │
│                                                              │
│  Batch / CDC                     Streaming                  │
│  - Airflow / Dagster             - Kafka / Kinesis           │
│  - Debezium / Fivetran           - Event Hubs                │
│  - Spark / Flink                 - Spark / Flink             │
│                                                              │
│  • Idempotent writes                                     │
│  • Schema capture & validation                           │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│                DATA LAKE STORAGE (OBJECT STORE)               │
│                                                              │
│  S3 / ADLS / GCS                                             │
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐     │
│  │  BRONZE      │ → │  SILVER      │ → │  GOLD        │     │
│  │  Raw data    │   │  Cleaned     │   │  Curated     │     │
│  │  Immutable   │   │  Standard    │   │  Business    │     │
│  └──────────────┘   └──────────────┘   └──────────────┘     │
│                                                              │
│  File Format: Parquet / ORC                                  │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│              TABLE FORMAT / TRANSACTION LAYER                 │
│                                                              │
│   Delta Lake / Apache Iceberg / Apache Hudi                  │
│                                                              │
│   • ACID transactions                                       │
│   • Schema enforcement & evolution                          │
│   • Time travel & snapshots                                 │
│   • Compaction & file optimization                          │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│                     METADATA & GOVERNANCE                    │
│                                                              │
│  Catalogs:                                                   │
│  - Hive Metastore / AWS Glue                                 │
│  - Unity Catalog / Apache Polaris                            │
│                                                              │
│  Governance:                                                 │
│  - RBAC / ABAC                                               │
│  - Row & column security                                    │
│  - Data classification (PII / PHI)                           │
│                                                              │
│  Observability:                                              │
│  - Lineage (DataHub / Amundsen)                              │
│  - Data quality checks                                      │
│  - Auditing & compliance                                    │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│                    COMPUTE / QUERY ENGINES                   │
│                                                              │
│  Batch / ETL             Interactive SQL                     │
│  - Spark                 - Trino / Presto                   │
│  - Flink                 - DuckDB                           │
│                          - Snowflake (optional)              │
│                                                              │
│  • Decoupled from storage                                   │
│  • Elastic scaling                                          │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│                 SEMANTIC & SERVING LAYER                     │
│                                                              │
│  - dbt / Metrics layer                                      │
│  - LookML / Cube                                            │
│  - Feature Store (ML)                                       │
│                                                              │
│  • Business definitions                                     │
│  • Reusable metrics                                         │
└───────────────┬──────────────────────────────────────────────┘
                ▼
┌──────────────────────────────────────────────────────────────┐
│                        CONSUMERS                             │
│                                                              │
│  BI Tools        Data Science        Applications             │
│  - Tableau       - Notebooks         - APIs                   │
│  - Power BI      - ML Pipelines      - Reverse ETL            │
│  - Looker                                                │
└──────────────────────────────────────────────────────────────┘

```

# Compare lakehouse vs traditional warehouse

Traditional Data Warehouse: A centralized, tightly coupled system optimized for:
- Structured data
- SQL analytics
- Strong governance
- Predictable performance
- Examples: Teradata, Oracle DW, SQL Server DW, Snowflake (modern but still warehouse-centric)

Lakehouse: A data lake with warehouse guarantees, built on:
- Object storage
- Open file + table formats
- Decoupled compute
- Multiple engines
- Examples: Delta Lake, Apache Iceberg, Apache Hudi (often with Spark / Trino / Databricks)

__A lakehouse is a flexible data platform that can behave like a warehouse, if you invest in metadata, governance, and tuning__

Comparison:
```
| Dimension    | Traditional Warehouse     | Lakehouse                                 |
| ------------ | ------------------------- | ----------------------------------------- |
| Storage      | Proprietary               | Object storage (S3/ADLS/GCS)              |
| Data types   | Structured                | Structured, semi-structured, unstructured |
| Schema       | Schema-on-write           | Schema-on-write + read                    |
| Compute      | Tightly coupled           | Decoupled, elastic                        |
| File formats | Proprietary               | Parquet / ORC                             |
| ACID         | Yes                       | Yes (via table formats)                   |
| Cost model   | Expensive, capacity-based | Cheap storage + pay-for-compute           |
| Openness     | Vendor lock-in            | Open standards                            |
| Streaming    | Limited                   | First-class                               |
| ML support   | Weak                      | Strong                                    |
| Governance   | Strong                    | Improving / depends on tooling            |
| Performance  | Very predictable          | Depends on tuning                         |
```

Data ingestion & modeling:
- Warehouse
  - Heavy upfront ETL
  - Star/snowflake schemas
  - Schema changes are painful
- Lakehouse
  - ELT-friendly
  - Raw → refined zones
  - Flexible schema evolution

Workloads:
```
| Use case               | Warehouse       | Lakehouse    |
| ---------------------- | --------------- | ------------ |
| BI dashboards          | Excellent       | Very good    |
| Ad-hoc analytics       | Good            | Excellent    |
| ML feature engineering | Poor            | Excellent    |
| Streaming ingestion    | Limited         | Native       |
| Data sharing           | Vendor-specific | Open formats |
| Data science notebooks | Awkward         | Natural      |

```

Choose a traditional warehouse if:
- BI is your primary workload
- Data is mostly structured
- You value predictability over flexibility
- Team is small or SQL-only
- Pros: Consistent query performance, Optimized for joins and aggregates, Easier to operate, Fewer moving parts
- Cons: Scaling is expensive, Pay for provisioned capacity, Storage is expensive

Choose a lakehouse if:
- You handle diverse data types
- You need streaming + batch
- ML is a priority
- Cost and openness matter
- You expect rapid evolution
- Pros: Massive parallelism, Object storage is very cheap, Compute can be turned off
- Cons: Requires tuning (partitioning, compaction), More components

# Explain Iceberg

Iceberg achieves ACID guarantees by treating metadata as the transaction boundary. Data files are immutable, and each table version is a snapshot pointing to manifests that describe data and deletes. This enables snapshot isolation, safe concurrent reads and writes, schema and partition evolution, and multi-engine interoperability — at the cost of requiring active metadata maintenance.

Iceberg turns object storage into a transactional database by managing immutable files with a metadata-driven commit protocol.

An Iceberg table is not a directory of files. It’s a metadata graph.
```
Iceberg Table
 ├── Catalog entry
 ├── Metadata files (JSON)
 ├── Manifest lists (Avro)
 ├── Manifests (Avro)
 └── Data files (Parquet / ORC / Avro)
```

The catalog (control plane) stores:
- Table name
- Current metadata file pointer
- Table properties
- Examples: Hive Metastore, AWS Glue, REST / Polaris

Metadata files
- Each metadata file represents a table snapshot and is immutable
- It contains: Schema(s), Partition spec(s), Snapshot list, Current snapshot ID, Table properties

Manifest lists & manifests:
- Manifest
  - Lists data files or delete files
  - Includes statistics:
    - Min/max values
    - Null counts
    - Record counts
  - This enables:
    - Predicate pushdown
    - File-level pruning
    - Fast planning

Data files
- Stored in object storage
- Immutable
- Columnar (Parquet most common)
- Iceberg never updates files in place.

ACID transactions (how writes work)
- Write flow
  - Writer reads current metadata pointer
  - Writes new data files
  - Writes new manifests
  - Writes new metadata file
  - Attempts to atomically update catalog pointer
  - If step 5 fails → transaction aborts safely
  - No locks on data files. Only atomic metadata swap.
- Multiple readers: Always safe.
- Multiple writers
  - Optimistic concurrency:
  - Conflict detection at commit time
  - Conflicting writes fail and retry

Schema evolution:
- Iceberg tracks schema by field IDs, not names.
- You can safely:
  - Rename columns
  - Reorder columns
  - Add / drop columns

