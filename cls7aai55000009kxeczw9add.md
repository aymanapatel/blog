---
title: "OLAP databases - A Primer"
seoTitle: "OLAP vs OLTP and the evolution of OLAP"
seoDescription: "OLAP, Duckdb, Oracle, Postgres, Row-orineted vs Column-oriented, dbt"
datePublished: Sun Feb 04 2024 09:11:40 GMT+0000 (Coordinated Universal Time)
cuid: cls7aai55000009kxeczw9add
slug: olap-databases-a-primer
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1707037589864/1d9b62cf-9fb7-435e-8623-35fb7f310bd4.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1707037855331/10685957-3d1a-4b41-97cb-f4e73cd99762.png
tags: databases, olap

---

We all are aware on the traditional relational databases such as MySQL, Oracle, Postgres etc. These are what we use daily for daily interaction through our CRUD (Create, Read, Update and Delete) apps. Whilst these are immensely useful and needful technology; there has been a new type of databases that are for generating insights on the transactions that have occurred in OLTP/traditional relational databases. That is where OLAP databases come in; they provide a historical context on the normal business operations.

OLTP can act as a analytical source of truth as is, but when you scale today's data, it becomes a complex system with huge amounts of data. These can lead to queries taking hours to run. Also, scalability and high availability at that scale becomes difficult, if a single-node architecture is used.

OLAP has gone through various iterations over the decades in order to solve the problem of complex queries, high availability, good querying performance.

Firstly, we'll look into the **Shared-\*** architectures (Shared Disk vs Shared Nothing Architecture)

# Shared Architectures

> Sharing means three things: **Compute**, **Memory** and **Storage**

## Shared-Everything Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707024036959/981f8044-b1f5-4edc-b8c6-6f7b2b368ba7.png align="center")

This is a single node architecture. Everything is shared. Example would be a UNIX OS where all things is in same machine. This is great for co-location but not good for fault tolerance and performance. Single server means single point of failure, hence ***no fault tolerance*.** All data in one machine without partitioning means ***performance degradation with larger data.***

Shared-Disk and Shared-Nothing provides some improvements in each of the criteria.

## Shared-Nothing Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707023870022/3285f406-43b8-404c-9c91-9b8658ebfd39.png align="center")

Each Database node shares:

1. CPU
    
2. Memory
    
3. Storage
    

These Nodes talk to each other via a shared **Network**. Data is split/partitioned into subsets across the nodes.

Data is fetched via POSIX API \[?\]

Shared-Nothing is harder to scale data capacity due to tightly-coupled storage layer. But these can have better performance due to co-location of data and compute per instance.

## Shared-Disk Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707023859340/6eae1261-f74f-4d49-aaa9-3c120370a006.png align="center")

Each Database node shares the compute layer:

1. CPU
    
2. Memory
    

The storage layer is decoupled and nodes can talk to the storage and with each other via Network layer(s).

Data is fetched via userspace API instead of a POSIX API such as `fread(0)`

Shared-Disk is easier to scale data capacity due to loosely-coupled storage layer. But these can have lead to higher computing resources utilization as the data needs to be moved from storage -&gt; compute layer and perform filter at compute layer.

## Row Oriented vs Column Oriented Stores

> [Paper Link](https://15721.courses.cs.cmu.edu/spring2023/papers/03-storage/p967-abadi.pdf)

One of key difference between OLAP and OLTP databases is how the data is stored.

In OLTP, as the data is queried based on IDs (Mostly), a single row/tuple is mainly the querying layer.

In OLAP, mostly the querying is based on Columns with several `GROUPBY` statements.

Hence, the traditional databases such as MySQL, Oracle, Postgres used row-oriented storage whilst OLAP databases such as MonetDB, Vertica use Column-oriented stores.

### Row-Oriented Store

Here data is stored as rows at storage layer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707032388586/55871359-4b33-48d3-ae2a-69c083f8a95b.png align="center")

For example

```plaintext
|ID| Name | Department |Role|
|10|Ram   | Devops     |SDE1| 
|20|Sam   | Security   |SDE2| 
|30|Aditi |Software Dev|SDE3|
```

In a traditional DB, ID#20 would be a query to get information on Sam. There could be some JOINs but the data in most cases would be for a particular row.

Row-oriented can be changed to column-oriented (as early OLAPs did by forking Postgres to be column-oriented); but these have the baggage of row-oriented metadata which does not bring major performance for Analytical workloads

Some implementation that were done to move row-oriented to column oriented-database

**Improving Row-oriented datastore to become Column-oriented**

| What | How | Disadvantage |
| --- | --- | --- |
| Vertical Partitioning | \- Connect column by adding position attribute to every column. | \- Row stores have heavy metadata info for every row. This increases space as this is not needed fro column scans |
| Index-only scans | B+Tree index for every column | Index scan for every read can be slower than traditional heap-file scan in vertical partitioning |
| Materialized Views | Create a temporary data fro faster access | Changing business need might make Materialized view largely irrelevant. |

### Column-Oriented Store

Same data is stored as columns at storage layer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707032406194/a3faab17-47ed-4b18-b35e-6ddf01bcb7de.png align="center")

For example

```plaintext
|ID| Name | Department |Role|
|10|Ram   | Devops     |SDE1| 
|20|Sam   | Security   |SDE2| 
|30|Aditi |Software Dev|SDE3|
```

In a OLAP DB, queries are mostly related to columns, like show me the list of users whose salary is &gt;$1000 who belong to the SDE2 role. Here, it makes sense to store data in columns so that the `GROUPBY` clauses are more efficient.

**Efficiency in Column-oriented**

1. **COMPRESSION**
    

It is easier to compress data in column-oriented data as similar data (Columns) are stored nearby. For example, if timestamp is present in column; a run-length encoded scheme can be used where the first timestamp can be stored as a Unix timestamp; whilst the rest of data can be stored as increment. This was lower the space cost as increments can be stored in less number of bits.

1. **Late Matrialization**
    

As data is located in separate places, the data is formed at the end as a lot of combining of columns takes place at later stage.

For aggregate, using Column-store makes more sense as a lot of unnecessary columns can be read in row-stores which are discarded in the query execution cycle.

All big data formats are looking into storing data in column stores.  
Examples for column-oriented file formats:

1. [Apache Arrow](https://arrow.apache.org/): Store data in columnar data memory format
    
2. [Apache Parquet:](https://arrow.apache.org/) Column-oriented data format for efficient data storage and retrieval
    
3. [Apache ORC:](https://orc.apache.org/) Columnar storage for Hadoop database
    

And examples for column-oriented data stores:

1. [Clickhouse](https://clickhouse.com/): Storing databases in column
    
2. [DuckDB](https://duckdb.org/why_duckdb#fast): Columnar-vectorized query execution engine
    

# OLAP over the decades

## 1990s - 1st first generation OLAPs - DATA CUBES

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707020229495/9086e02f-8df6-4348-b14a-14dfc303942b.png align="center")

Here the data from disparate sources are fetched and combined into a data cube.  
These are then aggregated which can then be queried for analyitcal purposes.

But this was created in the 1990s, where the data was simple and less granular than we have today. Now we have all sorts of data with varying types present in our databases. The simple Employee database with simple columns no longer exist. Hence, it lead to poor scalability and flexibility.  
Also, since data sources were from different sources, it lead to higher data duplication which in turn lead to higher data storage costs.

Examples:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707036188914/5177ab8a-dd1e-470c-bef8-aafb9021cd6f.png align="center")

| Tool | Note |
| --- | --- |
| MSFT SQL Server | Data Cube |
| Oracle | Data Cube |
| Terradata | OLAP from day1 |
| Sysbase | Data Cube |
| IBM DB2 | Data Cube |
| Informateica |  |

## 2000s - Second Generation Architectures

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707019243762/a3807a2c-e780-4cd5-baec-8baea8a66748.png align="center")

In 2000s, there was a new paradigm of ETL (**E**xtract, **T**ransform, **L**oad)

1. **E**xtract: Pull raw data from multiple sources. This could be databases, CSVs, JSON etc
    
2. **T**ransform: Transform data to be dumped to OLAP database. Transformation could be cleaning data, creating proper data model as per business need etc
    
3. **L**oad: Load into a database which can be consumed by business via tools such as PowerBI, Tableau, DOMO etc.
    

A lot of Data-warehouse came into picture such as MonetDB, Vermica.  
A hidden fact is most of these data-warehosues were forks of traditional Databases which were modified to improved coulmn-oriented analytical queries. This was discussed at ***Improving Row-oriented datastore to become Column-oriented*** section

Examples:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707036200559/392ccc38-66d4-4111-ac2d-40e4f408aeb0.png align="center")

| DB | Note |
| --- | --- |
| Netezza | Forks of Potgres |
| ParAccel | Forks of Potgres |
| MonetDB | Duckdb v1 was based on MonetDB. V2 is not MonetDb |
| Greenplum | Forks of Potgres |
| DataAllegro | Microsoft bought |
| Vertica | Fork of Potgres |

## 2010s and now: Rise of Decoupled OLAPs

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707020354226/ed42c162-b9ff-4e1e-abd1-9152c1d70ad6.png align="center")

1. Send Data to a decoupled Object Storage
    
2. Send Metadata information to a catalog/metadata store
    
3. Query Engines queries the catalog/metadata store to get the location and other information on how to query data.
    
4. Query Engine queries the object storage and pulls data.
    
5. Processes the data based on filtering criteria and sends result to client.
    

Modern OLAP has moved away from traditional ETL workflow to a ELT workflow, where the *transform* step happens alter load. [dbt](https://www.getdbt.com/) is one of the technologies that leverages ELT to transform data immediately after extracting.

Examples:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707036244312/d23247de-5f7c-4236-a7fb-397d11702646.png align="center")

| DB | What | Integration |
| --- | --- | --- |
| [Hive](https://hive.apache.org/) | DW system which is used with Hive-metastore. Not ideal for object storage which is part of modern OLAP | Integrates with Hadoop such as HDFS, Pig etc |
| Apache Drill | Schema-less Query engine for Hadoop, NoSQL and Cloud storage | Integrates with Hadoop ecusystem |
| Duckdb | In-porcess OLAP | Supports various file format such as CSV, Parquet JSON etc |
| Druid | Real-time analytics DB for batch and streaming data | Streaming such as Kafka |
| [Presto](https://prestodb.io/) | SQL query engine by Facebook | Integrate with various file format, OLAP DBs and visualisation tools |
| Snowflake | Written from groundup and is a whole platform for data engineering including hosting, querying, RBAC etc | Integrates with everything |
| Pinot | Realtime distributed OLAP datastore from Linkedin | Integrates with Kafka, S3, Tableau, Presto etc |
| Redshift | Built on top of ParAccel | Integrates nicely with AWS ecosystem |
| Apacka Spark | Data analytics engine to do high-volume data processing | Integrate with object stores etc |
| Google Bigquery | Google's OLAP | Integrate with GCP |

# ETL vs ELT and the DBT Approach

1\. **ETL**

The three phases as discussed in the 2000s database section can be represented here:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707035613754/68e3ecc1-800c-4a08-852b-10c8dd4a7c56.png align="center")

1. **ELT**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707035630304/668ce0af-5ff3-44e7-8191-d5b153d0a0af.png align="center")

1. DBT: Implementing ELT in real-world
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707035650948/5b0a6bf4-3b69-4091-a1de-d115acda7876.png align="center")

### Advantages of ELT vs ETL

1. Versioned Control workflows: As in ELT; data transformation requires a feedback with data load; it is possible to allow VCS and CI/CD workflows on the data which is a common practice in Software engineering and Agile teams
    
2. Faster feedback loop: As data is transformed on the fly, we can get a better feedback on insights from data, instead of waiting for a cron job that executes the data dump into datawarehouse.
    

# Push vs Pull between Data & Query

There are a couple of ways to execute query in OLAP ecosystem. These are

1. **Push Query to Data**
    
2. **Pull Data to Query**
    

## 1.Push Query to Data

In the earlier days, networking and data fetching was slow and expensive. So idea was to send query directly to the OLAP database system. Then data (based on executed Query) was sent back to client over the network.

## 2.Pull Data to Query

Now we have modern OLAP with Shared-Disk architecture where the compute cannot be possible in shared-disk-not-compute architecture. Also, the network and data fetching (thanks to SSDs) speeds have improved. This allows of moving the data to the **query engine**(as discussed earlier) for processing.

Modern OLAP does the combination of 1. and 2. in context of what type of dat a workload, how frequent data are accessed (available via data querying statistics); decisions are made whether to bring query to data; or data to query.

# Conclusion

Hope you learned the need of OLAP databases and why there has been a degree of change for every decade in the OLAP world.

The following the references without which this could not have been possible:

# References

1. %[https://www.youtube.com/watch?v=7V1oi_8uvuM&list=PLSE8ODhjZXjYzlLMbX3cR0sxWnRM7CLFn&index=2] 
    
2. [Paper Link](https://15721.courses.cs.cmu.edu/spring2023/papers/03-storage/p967-abadi.pdf): Column-Stores vs. Row-Stores: How Different Are They Really?