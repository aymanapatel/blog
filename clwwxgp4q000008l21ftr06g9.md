---
title: "Dr Postgreslove"
seoTitle: "Dr Postgreslove"
seoDescription: "How I stopped Worrying and Love Postgres"
datePublished: Sun Jun 02 2024 02:33:24 GMT+0000 (Coordinated Universal Time)
cuid: clwwxgp4q000008l21ftr06g9
slug: dr-postgreslove
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717273342527/facff9ea-bf0a-43b4-b018-64d5173700d7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1717273358747/b562e01f-693f-4f9b-a54c-28b0ffc4a8ca.png
tags: postgresql, databases

---

Data is not static. It is dynamic in ways that were not dreamed of by Ted Codd when he wrote about relational databases "[Relational Model of Data for Large Shared Data Banks](https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf)", which are the bedrock of most data systems today. All computer science concepts converge on databases. Distributed systems for highly available databases, filesystems for efficient and compact data storage, compilers for translating the SQL queries.

The inspiration of this blog was a Youtube video, [“Wait... PostgreSQL can do WHAT?”](https://www.youtube.com/watch?v=VEWXmdjzIpQ). The video just opened my ways on what databases can do and especially what Postgres can do. Whilst I was watching the video, I realised if Postgres can do all these things that we overlook, it would be hilarious that it could run Doom. Guess what? It can!

# Postgres eating NoSQL's lunch

We have all seen the 4 types of NoSQL (I would like to call them the 4 horsemen):

1. Key Value: Redis,
    
2. Document-oriented: Mongo, CouchDB, Amazon DynamoDB
    
3. Column Family: Cassandra, HBase
    
4. Graph: Neo4j
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717303011754/433da79d-a900-45c9-9182-b584f44f5974.png align="center")

Funnily, Postgres has a solution for all these NoSQL horsemen!

## Key Value store

KV are basically a simple data structure where there is a key by which you query your values that could be of many data structures such as string, map, sets, hashes etc. KV stores such as Redis and Memcached became popular in the 2010s for caching use-cases.

Postgres has an extension called `hstore,` which enables the creation of a data type `hstore` that can be utilized as a K-V column.

1. Create `hstore` extension
    

```sql
CREATE EXTENSION hstore;
```

2. Create a table with `hstore` column
    

```sql
CREATE TABLE postgres_supremacy (
	id serial primary key,
	extension_name VARCHAR (255),
	extension_attr hstore
);
```

3. Add data (check the script for more information!)
    

```sql
INSERT INTO postgres_supremacy (extension_name, extension_attr) 
VALUES 
  (
    'Hstore', 
    -- Attributes
    '"postgres_version_compat" => "from postgres 8.3",
     "released" => "2008",
     "documentation"  => "https://www.postgresql.org/docs/current/hstore.html"
  );
```

## Column Family database

Column family or column-oriented databases are typically used in Big Data applications.

Postgres has a couple of solutions for this

1. [Citus](https://www.citusdata.com/): A HTAP database that has capability of being. Column-oriented database, Citus will be discussed below.
    
2. [Hydra](https://www.hydra.so/): A fork of Citus that is focused purely on being a columnar analytics database.
    

Hydra has [data ingestion](https://docs.hydra.so/concepts/batch-ingestion-and-data-streaming) capabilities, can work with data transformers such as [dbt](https://docs.hydra.so/organize/data-hygiene/dbt-transformation-and-model-management), and provide insights using BI tools such as [PowerBI](https://docs.hydra.so/guides/saas-retention-tracking-and-churn-with-powerbi)

## Graph

Graph databases like Neo4j have not become mainstream. But at least, there is a Postgres extension(more on it later!) which is from the Apache foundation called **AGE**.

[AGE](https://age.apache.org/) is an extension connects to Postgres and also supports the typical Graph Database querying language called Cypher. (You can also utilize SQL for querying.)

Finally, it also had a visualisation tool to show the data points and connections in the Graph Database.

## Document Oriented

In the early 2010s, a new kid on the block had come from the ashes and that was MongoDB. It was meant to be “scalable” and “flexible” as it worked on collection. If you want to simplify it, you could think of it as a flexible JSON that gets stored in the database. Initially, Mongo sacrificed speed for data durability and, as it became more relevant in big applications, the lack of ACID compliance became an enormous blow to Mongo. Mongo therefore started added transactions and changed their storage from mmap to Wiredtiger.

Postgres (and other databases) looked into supporting JSON as a data type. Postgres has both JSON and JSONB(B stands for binary); along with neat helper functions for querying keys in the database. I wrote more on this [here](https://aymanace2049.hashnode.dev/json-in-traditional-databases).

So in the end, Mongo changed its architecture to be more like a traditional database and on the other side, traditional databases added support for JSON. The pendulum has done its full swing; I guess!

All the 4 alternatives show how Postgres always have an answer to the new database paradigms. It might sound like Postgres is becoming a “jack of all trades, master of none”; but one can argue that having wrangled data systems for each type of database can be an overhead in terms of overall performance and cost of maintenance.

# Modern DBs built on top of Postgres

In this section, we’ll discuss 5 databases with varying use-cases that are built on top of Postgres.

# Supabase

Supdabase is marketed as an “OSS alternative to Firebase”, which is quite an undersell. It does a lot of what Firebase does; but its key advantages are:

1. Self-hostable Postgres. If you want to move away from Supabase, you can self host the Postgres database itself.
    
2. Postgres extensions and features: Since it is Postgres, you could use Postgres extensions and features which makes you architecture to be very flexible.
    

## Supabase Features

> [Supabase](https://supabase.com/docs)

1. [Database](https://supabase.com/docs/guides/database/overview) Full Postgres and its extensive extensions. What Supabase provides is a great GUI for managing stuff like database tables, Schema visualizer, SQL editor, etc.
    

![Supabase web editor](https://cdn.hashnode.com/res/hashnode/image/upload/v1717303033393/46890f13-b785-4e56-b155-d3dc78a4e676.webp align="center")

1. [Authentication](https://supabase.com/docs/guides/auth) Supabase provides authentication and authorization with help of Supabase Client SDKs for UI frameworks such as React, Kotlin, Next.js, Flutter etc. Supabase uses JWTs for authentication. As it is based on Postgres, Supabase also has **Row level security** for authorized access at the row-level. Supabase provides social logins such as for Google, Apple, etc.
    
2. [Storage](https://supabase.com/docs/guides/storage) S3 compatible object storage which you can use to store images, CSVs and other files
    
3. [Edge Functions](https://supabase.com/docs/guides/functions) Based on Deno, it allows to write globally distributed functions to execute small but fast one-off functions such as Payment, etc
    
4. [Real-time](https://supabase.com/docs/guides/realtime) communication between clients
    

* Broadcast: Send messages between clients that use Supabase
    
* Share state: Track, share and synchronized state between clients that use Supabase
    
* Database Changes: Listen to Database changes in real time
    

4. [Vector](https://supabase.com/docs/guides/ai) Creates vectors from data and then proceeds to create embeddings for AI apps. It uses `pg_vector` to create a vector store.
    

# Neon

> [Neon](https://neon.tech/docs/introduction)

Neon is a serverless Postgres which is a highly distributed, serverless database that can scale down to zero. It is inspired by Amazon Aurora which is one of the mainstream databases solutions that was architected to **separate compute and storage**.

Neon replaces the Postgres storage layer with its own storage engine. Neon storage engine consists of:

![Neon Architecture: Fetched from Github discussion](https://cdn.hashnode.com/res/hashnode/image/upload/v1717303060518/795fc5ef-555f-41c5-8577-65acf9948249.jpeg align="center")

1. Safekeepers: As the name suggests, it is safekeeping something. And that something is **WAL**. It stores WAL that comes from the compute layer/node until page server saves data in all nodes.
    
2. Page Server: It comes in between the Safekeeper and S3 durable store and is responsible for writing data to disk(S3).
    

You can read more on the Neon architecture from this [Github discussion thread](https://github.com/neondatabase/neon/discussions/1770) and the \[[Jack Vanlightly’s blog](https://jack-vanlightly.com/analyses/2023/11/15/neon-serverless-postgresql-asds-chapter-3)

# Yugabyte

> [Yugabyte doc](https://docs.yugabyte.com/)

In the mid to late 2010s, there was a new type of databases that came into the picture, which was NewSQL which was meant for distributed ACID-compliant transactional database. Google’s Spanner was one of the pioneers in building a globally distributed transactional database. Many databases such as Azure Cosmos DB, CockroachDB, FoundationDB and finally Postgres-based Yugabyte came into the picture.

## Yugabyte Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717303087908/ba4f96b0-f529-45b9-bb8a-0b710c618904.png align="center")

1. YB-Master: Holds metadata of the database system, such as tables, user permissions, etc. It is also responsible for load balancing of data which are not yet replicated. It is part of the Raft group where the leader is elected as part of the Raft protocol.
    
2. YB-Tserver: Tserver is responsible for all I/O operations which are carried out in tablets at the DocDB layer.
    
3. Query layer: The querying layer has support for both SQL and Cassandra Query language - YSQL: SQL API which uses Postgres query engine - YCQL: Similar to Cassandra Query language The query layer is responsible for parsing, analysing query, query planning for optimum query execution and finally executing the query.
    

* Storage: DocDB is the storage engine that is used here. It is based out of [Rocks DB](http://rocksdb.org/) (a [Log-Structured Merge Tree](https://docs.yugabyte.com/preview/architecture/docdb/lsm-sst) KV store from Facebook). The tables are split into tablets for partitioning.
    

# Timescaledb

> [Link](https://www.timescale.com/)

Database are not just a sequence of rows and columns with varying types. There are some data that are high-volume datasets that are temporal in nature(time-series based). These data sets can become quite large, with high writes. Optimising for reads and writes is very different in Time-Series data when compared to traditional data workloads. High cardinality (column data with different values) is one bottleneck for read performance.

One of the abstractions created by TimescaleDB on top of Postgres is the **hypertable**. All inserts and reads get pushed to a single hypertable as a user. Underneath, TimescaleDB handles partitioning across nodes to optimize write throughput and read performance.

# CitusDB

With the rise of Data Warehouses for OLAP use cases in the 2000s, a lot of Data warehouse technologies were created. Many of these were forks of Postgres which changes to be compatible with OLAP workloads by making Postgres somehow column-oriented. Examples include ParAccel(AWS forked it into Redshift), Netezza(IBM acquired), Vertica and Greenplum (VMware).

Citus is one of databases that is a mixture of both OLTP and OLAP, called **HTAP database**. It was acquired by Microsoft in 2019 and has been part of the Azure ecosystem.

So there you have it. Postgres is one of the most exciting databases with various companies building platforms on top of Postgres!

# Postgres Extensions

# Postgres Extensions

Postgres has the ecosystem to be extended for different use cases. The extensions are hookable to many layers of the stack in Postgres itself, which makes it convenient for the Open source community to create extensions that serve a particular niche purpose. Like with the rise of LLMs, there is a new breed of Vector databases that install Vector data types along with embeddings. Postgres has an extension for that called `pg-vector`.

In this section, we’ll see how to see those extensions, some useful extensions and also how to create one.

## Finding extensions

There are basically 2 ways to finding extensions.

One is the traditional way to find in Postgres with the help of a SQL command

```bash
SELECT * FROM pg_catalog.pg_extension;
CREATE EXTENSION <extention-name>;
```

Couple of issues with the above approach is the installing new packages and discoverability of these packages. Installing new packages that are not part of your Postgres installation can be a little [tricky](https://docs.yugabyte.com/preview/explore/ysql-language-features/pg-extensions/install-extensions/). You need to install the shared library files (`<name>.so`), SQL files (`<extention-name>.sql`) and control files (`<extension`.control) which can become a hassle.

Like in Linux you have registries/marketplaces for discovering apps (such as Flathub, Snap store etc); there is a package installer for Postgres called [Trunk](https://pgt.dev/)

To use Trunk, you need to first install Rust Cargo toolchain:

```bash
curl https://sh.rustup.rs -sSf | sh
```

Then you can install the Trunk CLI running cargo command (Rust’s package manager)

```bash
cargo install pg-trunk
```

Finally, you can install the Postgres extenstions by running the `trunk install` command:

```bash
❯ trunk install pgmq
Using pg_config: /usr/bin/pg_config
Using pkglibdir: “/usr/lib/postgresql/15/lib”
Using sharedir: “/usr/share/postgresql/15”
Downloading from: https://cdb-plat-use1-prod-pgtrunkio.s3.amazonaws.com/extensions/pgmq/pgmq-0.5.0.tar.gz
Dependencies: [“pg_partman”]
Installing pgmq 0.5.0
[+] pgmq.so => /usr/lib/postgresql/15/lib
[+] extension/pgmq--0.5.0.sql => /usr/share/postgresql/15
[+] extension/pgmq.control => /usr/share/postgresql/15
```

## Popular Extensions

1. PostGIS
    
2. pgmq
    
3. FDW
    
4. hstore: Already discussed
    
5. pgvector
    
6. pgvector
    

### PostGIS

A very well known extension for inserting geographical data. Mongo also has GeoJSON for storing geographical data [Mongo GeoJSON](https://www.mongodb.com/docs/manual/reference/geojson/).

### pg\_stat\_statements

Query statistics such as rows, index scanned etc.

### Foreign Data Wrapper

Standard way of retrieving/inserting data from other data sources, be it different Postgres server database, other databases such as Oracle/Mongo/SQL server

### `pg_cron`

Run periodic jobs in Postgres. Use cases of `pg_cron` can be cleaning old data, partitioning using `pg_partman` etc

### `pg_partman`

Partition management in Postgres. Can be used with `pg_cron` for automatic partition creation and management

### `pg_vector`

Store embeddings from LLM

### `pg_crypto`

An extension that provides cryptographic functions taht can be applied to data. These functions include :

1. Create Symmetric Key encryption
    
2. Generate passwords
    
3. Hash
    

For transparency in security, many traditional databases such as Oracle, Microsoft’s SQL Server and IBM support secure *data at rest* by leveraging **Transparent Data Encryption** to encrypt database files Postgres does not support Transparent Data Encryption (supported in forks of Postgres)

### Using Postgres extension

A neat tool I have found is [Tembo](https://tembo.io/) where you can install a lot of Postgres ecosystem with a Web dashboard. You can import the Postgres database in your GUI of choise (DBeaver, DBStudio, Tableplus etc)

# Parting thoughts

If you have read through this blog, thanks for your time.