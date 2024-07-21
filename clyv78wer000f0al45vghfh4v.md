---
title: "Postgres FDWs: Unlocking external data sources into Postgres"
seoTitle: "Postgres Foreign Data Wrappers"
seoDescription: "A blog on how Postgres Foreign Data Wrappers can be leveraged for loading all kinds of Data into Postgres"
datePublished: Sun Jul 21 2024 06:51:09 GMT+0000 (Coordinated Universal Time)
cuid: clyv78wer000f0al45vghfh4v
slug: postgres-fdws-unlocking-external-data-sources-into-postgres
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721544618901/0a5e3a04-fc15-42fd-97b9-47dd98f38e64.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1721544636576/3eadb752-98a8-46d5-b27c-e0d89871d643.png
tags: postgresql

---

Modern data pipelines consist of moving data from one format to other, from one source to another. Tools like ETL exist to facilitate this kind of Data movement.

# FDW Use Cases

## Zero ETL overhead

Most popular ETL tools include Airbyte, Apache Airflow, Apache Nifi. These tools are versatile that allows integration between all combinations of Data sources and sinks. But ETLs have some issues such as:

1. It is async in process so data will always lag based on the ETL job run which is typically nightly/weekly.
    
2. Maintenance overhead as it is a separate system. There is also a cost component that is also thst needs to be compared.
    

Even though ETL have overheads, it is sometimes required to do complex Joins, filtration, materialized views. ETLs are still the go-to tool for this use case.

In the Postgres FDW world, the big data file formats such as Parquet, ORC are supported as Postgres FDWs

# List of FDW

| Foregin Data Wrapper | Feature |
| --- | --- |
| File FDW | Simplest FDW |
| Mongo FDW | Read Write from Mongo |
| Kafka FDW | Read Write from Kafka |
| MySQL FDW | Read Write from MySQL |
| Oracle FDW | Read Write from Kafka |
| JDBC FDW | Read Write via JDBC API |
| TDS FDW  
(Sybase/Microsoft SQL Server) | Read Write from Sybase or Microsoft SQL Server |
| Postgres FDW | Read Write from remote Postgres database |
| Prometheus FDW | Read Write from Prometheus Metrics |
| Redis FDW | Read Write from Redis |
| ORB FDW | Read Write from ORB |
| Parquet\_S3 FDW | Read Write from Parquet Files |
| Duckdb | Duckdb is an promising in-memory OLAP database |
| Clickhouse | Read Write from Clickhouse |
| Clerk FDW | Clerk is an AuthN/AuthZ user management platform |
| OGR FDW | Read Write from Kafka |
| Posgtres Log FDW | Read Write from Posgtres Logs |
| Pgbouncer FDW | Read Write from Pgbouncer |

## Importing from different data sources

In the age of large enterprises, thousands of Microservices, thousands of teams with apps that have different databases, it is sometimes necessary to move the data around.

In Postgres, there are FDW for many database systems :

1. Oracle FDW
    
2. TDS database (Sybase and Microsoft SQL server)
    
3. Mongo FD
    
4. Postgres FDW to import from remote Postgres server
    
5. MySQL FDW
    
6. JDBC FDW
    
7. CSV FDW using `file_ftw`
    

# Building blocks of FDW

## Step 0: Installing the FDW extension

## Step 1: Server

```sql
--- 1. Create Server
CREATE SERVER <custom_server_name> FOREIGN DATA WRAPPER <fdw_name>;

-- fdw_name is the name of the extenstion that you have installed
```

## Step 2: User Mapping (Optional)

This is optional yet important thing to do when you are using FDWs. As for any database, you need to create a User Mapping to make a 2-way binding on how the Local Postgres Server's users can talk to Remote Postgres Server's users.  
For File FDW, it is not required as the imprt method is either a local file or a file hosted on a public server.

But User Mapping for other use-cases is very important and to be done in a way that is secure.

## Step 3: Foreign Table

```sql
CREATE FOREIGN TABLE <table_name>
(
   --- Table Schema
    id TEXT,
    userId TEXT,
    userName TEXT,
   --- ...
 
)
SERVER <custom_server_name> 
OPTIONS (
    --- Different for each foreign data wrapper
);
```

# File FDW

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721533153387/c9af175d-1e91-4e3a-8162-d1cfa91d560c.png align="center")

File FDW works by using [COPY FROM format](https://www.postgresql.org/docs/current/sql-copy.html) which is part of Postgres.

1. Creating Server
    

```sql
--- 1. Create Server 
CREATE SERVER fdw_files FOREIGN DATA WRAPPER file_fdw;
```

2. Creating a Foreign Table
    

```sql
CREATE FOREIGN TABLE public.covid_staging
(
    fips TEXT,admin2 TEXT,
    province_state TEXT,country_region TEXT,
    last_update TIMESTAMPTZ,
    lat FLOAT,lon FLOAT,
    confirmed INT,deaths INT,recovered INT,
    active INT,combined_key TEXT
)
SERVER fdw_files 
OPTIONS (
 
    program 'wget -q -O - "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/03-26-2020.csv"', 
    -- filename for local filesystem. program for external file system
    format 'csv',
    header 'true'
    -- Other options. 
    --- 1. delimiter
    --- 2. quote
    --- 3. escape
    --- 4. null
    --- 5. encoding
);
```

Here `program` is used if you want to import from another system.  
If you have a data from the local Postgres Server, you can use `filename : <path-to-file>/mydata.csv`

You can also set `delimiter`, `quote` , `escape`, `null` and `encoding` for parsing the text input.

3. Read the data that is imported
    

```sql
SELECT * FROM public.covid_staging;
```

The data can be seen by running the above `SELECT` command:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721538296033/c862c155-5d4e-4168-8060-72a1bf77c187.png align="center")

# Postgres FDW

This is trivial as most Postgres distributions have `postgres_fdw` enabled. Have to follow the building blocks of FDW:

1. Creating a **Foreign Server**
    
2. Creating a **User Mapping**
    
3. Creating a **Foreign Table**
    

# Non-Postgres FDW

> MySQL FDW example

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721533207408/bdf49985-72b5-4a57-9316-f7115deb0fb4.png align="center")

> I am using [Tembo](https://tembo.io/) database for installing extensions. It is very intuitive way of installing Postgres extensions. They have a GUI which internally uses [Trunk](https://pgt.dev/) (Rust-based Postgres Extension manager).

Similar to the building blocks of FDWs, we are:

1. Creating a **Foreign Server**
    
2. Creating a **User Mapping**
    
3. Creating a **Foreign Table**
    

```sql
-- 1. create Foreign Server
CREATE SERVER mysql_server
         FOREIGN DATA WRAPPER mysql_fdw
         OPTIONS (host '<my-host>', port '<port-number>');

-- 2. create User Mapping
CREATE USER MAPPING FOR postgres
    SERVER mysql_server OPTIONS (username '<username>', password '<password>');

-- 3. create Foreign Table
CREATE FOREIGN TABLE warehouse
(
 warehouse_id      INT,
 warehouse_name    TEXT,
 warehouse_created TIMESTAMP
)
SERVER mysql_server
         OPTIONS (dbname 'defaultdb', table_name 'warehouse');      
INSERT INTO warehouse values (6, 'UPS', current_date);               
SELECT * FROM warehouse ORDER BY 1;
```

The result can be seen:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721538137314/c0b04187-984f-4185-9097-46139184af2b.png align="center")

> Tembo does not support `mysql_fdw` 's Pushdown feature which was a recent addition.(Remote MySQL -&gt; Postgres reading); hence cannot show Postgres screenshot.

# Parquet FDW

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721538603958/29b5315c-113a-4f57-927d-2ce510f89c16.png align="center")

A lot of dara pipelines have parquet files for efficient data storage. These can also be imported via the `parquet_s3_fdw`

# Closing Thoughts

Depending on your scale, expertise and requirement, you can leverage Postgres as the source of truth from various data sources.

Keep in mind, that data impedence will still be present. And large-scale ETL might still be the way to go.

But it is always good to know what you get in the Postgres world.