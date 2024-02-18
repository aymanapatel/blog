---
title: "Change Data Capture - Capture Data In Motion in your microservices"
seoTitle: "Change Data Capture - How to track Data in Motion"
seoDescription: "Change Data Capture - How to track Data in Motion by using microservice patterns such as Outbox, Saga, Strangler Fit"
datePublished: Sun Feb 18 2024 15:37:44 GMT+0000 (Coordinated Universal Time)
cuid: clsro8wek000209jo4mnfbhxa
slug: change-data-capture-capture-data-in-motion-in-your-microservices
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1708270507807/0f6c5aa5-b8ca-4a84-8d35-6c389b27d3ab.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1708270490461/9388e30e-118f-4b11-ab0a-b850eb52097a.png
tags: microservices, databases, change-data-capture

---

Data is the new oil. It keeps on coming and difficult to manage where there are multiple sources. Also, the database is not just a table as it used to be. Their are a multiple of formats and use-cases for data. It could be traditional Relational Databases, it could be data warehouse for analytical process, it could be a short-lived cache for performance improvement, it could a search index for optimising Database query reads, it could be a event notification that needs to be sent out to customers and the list goes on.

Basically, all these are changes or deltas that need to be sent from source to destination. That is where **Change Data Capture** (CDC) comes in.

# What and Why of CDC

## What is CDC?

> CDC is a paradigm to track changes in database (create, update, delete operations) and send those data to destination so that data is in-sync with real values

## Why CDC?

The concept of data is not confined to a single table or a schema. It evolves, it moves around for various applications. Also the format of the said data is non-uniform.

Traditional Databases are good for Client-Server applications where APIs expose CRUD operations for the database. Data is usually noramalized form in various tables. For analytical, OLAP databases are good which requires to be in denormalized forms. Since there is a difference/delta in the schema; we need a CDC or a ETL pipeline that is able to convert the data into the appropriate forms. Another example is for Data Another example would be sending data to cache so that cache can be invalidated for stale values and inserted for updated values. RDBMS work with tables while Cache is a Key-Value Store. Again, we need a way to translate RDBMS's table-type data to Cache's KV store. ([Blog on how to do this](https://debezium.io/blog/2018/12/05/automating-cache-invalidation-with-change-data-capture/))

# Pull vs Push Based

## Push

Here the source database does most of the work. It implements the capture changes, transformation, send update to target services. Disadvantage is that it does not consider the fallacies of distributed computing such as *Network is not unreliable*, hence the destination will have inconsistent state.

## Pull

Here there the target services takes the job of pulling from the data source. Drawback is that it is a new paradigm with a lot of plumbing required to make this work.

# CDC Implementation

## Timestamp-based

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267265764/a333d387-0ebd-48bf-834d-9108f9c36e67.jpeg align="center")

`LAST_UPDATED_BY` column are used by downstream services to get the latest information. It is dead-simple. but there are a couple of major drawbacks:

1. Soft deletes (Deletes which are available as row but not persisted to disk) is possibe, but not `DELETE` operations that are persisted.
    
2. Target systems scanning the database is not a free-lunch process. It adds overhead to source database.
    
3. Data might be outdated as in-progress transaction can lead to data inconsistencies in target systems.
    

## Trigger-based

Most Relational Databases have **TRIGGER** functions. These functions (aka Stored Procedures) that can be initiated to a database event (`INSERT` , `UPDATE` etc) Issue with trigger-based is the overhead on database as every write to database requires 2x resource (1 for update table, other for trigger actions)

## Log based

> Debezium Used this approach

A pull-based mechanism that reads the logs emitted by DB. This could be Write-Ahead-Logs that are saved in Databases

* BinLog for MySQL
    
* Replication Slots for Postgres
    
* REDO Log Oracle equivalent
    

# Libraries

## Kafka connect

> [Documentation Link](https://docs.confluent.io/platform/current/connect/index.html)

It is a middle-ware to connect Kafka with external systems such as Databases, Data Warehouse, Caches (K-V stores), file systems (S3 etc), and search indexes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267115300/b898af0b-a598-45fe-a985-881da830cac7.jpeg align="center")

Kafka Connect can take data from **source connector** as an ingest, and deliver this data to **sink connector** such as HDFS, Indexes such as Elasticsearch etc **Source and Sink Connectors** are configured so that the data ingestion and egress is possible based on those configuration.

## Debezium

> OSS CDC platform with a lot of connectors for various different type of data sources.

### Debezium Features

* Reads Transaction Logs on database (WAL; BinLog MySQL, REDO Log Oracle & Replication Slots Postgres)
    
* Outbox pattern support (Explain Outbox pattern)
    
* Web based UI
    
* Purely OSS
    
* Large production deployments
    

### Debezium Deployment Options

#### Kafka Connect

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708270544728/ebc4bd41-fd2a-40a5-bc81-6aa274804052.png align="center")

`DS -> Kafka Connect (Debezium DB-specific connector) -> Kafka`

#### Debezium Embedded Engine (Java only)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708270565121/d5d05bc5-5a12-460a-af64-3230f184478d.png align="center")

`DS -> Debezium Embedded Engine (Spring, Quarkus) (Debezium DB-specific connector) -> WHatever you want`

#### Debezium Server

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708270572719/491dd90a-43b7-4800-a4f8-340d59fe1623.png align="center")

`DS -> Embedded Engine Source Connectors (Debezium Server) -> Kinesis, Pulsar, Google Pub/Sub`

# CDC Patterns

## Transactional Outbox Pattern

### Problem Statement

Consider you have a main microservice (`Insurance Claim μs`) that has a couple of downstream services (`Insurance Settlement μs` & `Credit Score μs`)which are dependant on the aforementioned `Insurance Claim μs`. The downstream APIs consume the upstream `Insurance Claim μs` via a message broker (Could be Kafka, Pulsar, RabbitMQ etc). In Distributed Systems, we have 8 fallacies of Distributed Computing (Read [wiki](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)); first one is api in this scenario i.e. **Network is Reliable**. Consider the `Credit Score μs` Kafka Topic/whatever message broker you are using is down for some reason! In such a scenario, we don't want 2 out of 3 μs to be in consistent while the Credit Score μs to be inconsistent. Hence, **Outbox Pattern** to rescue us in this conundrum. We need to send the events *reliably!* Reliably means that it should be consistent with the Event that is being produced by the upstream `Insurance Claim μs`. And it should handle system crashes such as Server, Message Broker, database, N/W packet loss etc. Basically, you got to assume Murphy's Law!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267786276/12e721ed-d453-445f-953d-54d7894aa689.png align="center")

One question for those aware of [2-Phase commit](https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html)(aka XA transactions)would ask as it is also concerned handling consistent and atomic writes across different databases. The issue with 2PC is that it requires similar database and message broker setup and to be able to support 2PC out-of-the-box. Many message brokers and NoSQL cannot support 2PC, hence we are going up with this pattern.

If at-least-once semantics are present it will lead to dual-writes!

## Solution

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267822070/ab1c2029-e539-40c2-be40-49d1092dc7c3.png align="center")

Since we do not have possibility of using 2PC (XA transaction), we need to find an alternate path of making the event messages to be sent reliably and persisted in the downstream services.

The basic fundamental of this is that we do not directly add the events to the Event Bus/Message Broker. We add an **Outbox Table** which stores the messages to be sent. The table would look something like this:

| SpanId | AggregateType | PrimaryId | EventType | Payload |
| --- | --- | --- | --- | --- |
| dd12 | InsuranceClaim | 212 | InsuranceClaimCreated | `{id:212,...}` |

The **Change Data Capture** server acts like a message relay that read this Outbox table and sends to Message broker which then passes on the event to downstream.

(Postgres does not need as they have `pg_logical_emit_message()`)

The below diagram shows the steps

1. Add to **Insurance** Table
    
2. Insert to **Outbox** Table with the IDs, payload, Event Name (and other columns)
    
3. **CDC Server** (like Debezium) reads the ***Outbox*** table
    
4. CDC Server send this Event to **Message Broker** (like Kafka)
    
5. Sends data to **downstream services and their associated Databases**
    

This ensures:

1. Event messages are sent if and only if the data transaction is completed in source microservice.
    
2. Message Ordering is maintained at the Message broker as a single topic partition failure does not create a delta of inconsistent data.
    

Drawbacks of this pattern:

1. Additional resource and tables required.
    

## Strangler Fit Pattern

![Phase 1](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267851640/8475e11b-6756-4c7e-96d6-d87fa3c58b70.png align="center")

It is a pattern to allow for incremental migration from a legacy service to a greenfield new microservice. Here CDC is a very useful pattern to incrementally migrate the services from legacy to new.

In this pattern, the first step is to setup the new service in such a way that the new services's Database is able to read from the old database. CDC along with message broker can allow the writes to go from the legacy database to the new service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267893967/d0d03680-ab09-4aff-a128-d984c6978d5b.png align="center")

Here the user will write to legacy but will be reading from the new service. This is achieved by load balancing the requests so that the reverse proxy routes all writes to the legacy and the reads come from new service. Once the new service is at feature-parity with the legacy; then the reverse proxy can route the writes also to the new service. As the bonus step, reverse-CDC can also occur to insert the data from the new service to the legacy database via the reverse CDC Message broker setup.

## SAGA Pattern

> Solve for Long Running Service Transactions

Sometime the microservices are not so simple to execute. They might be some transactions that take a lot of time and a simple *Request/Response* API is not enough to bring Database consistency.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708267912781/183c2359-51d5-4011-bdfa-010261be0f53.png align="center")

Here the upstream `Insurance Claim μs` is dependent on Insurance Settlement and Credit Score service which is a slow process which can take more than n2 business working days to fulfil. Here SAGA pattern comes into the picture. Unlike **Outbox Pattern** where the upstream changes needed to be propagated to downstream reliably; here the downstream needs to propagate its state to the upstream service `Insurance Settlement μs` & `Credit Score μs`.

The pattern works as follows:

1. **Insurance Claim μs** sends the updates to the message broker
    
2. Message broker **fans-out** the event to downstream services **Insurance Settlement μs** & **Credit Score μs**
    
3. and 4. **Insurance Settlement μs** & **Credit Score μs** propagate their results to the message broker via a CDC server.
    
4. **Insurance Claim μs** receives both responses and makes a decision.
    

Drawbacks:

1. Without Observability, it is difficult to debug/trace the transactions across service, database and message broker.
    

# Resources

* Everything Gunnar Murling
    
* [Fanout with Postgres and CDC - Neon Blog](https://neon.tech/blog/fan-out-postgres-changes-using-debezium-and-upstash-redis)
    

# Resources (for me; not for blog)

* [Reddit on blog: CDC is anti-pattern and we still need it](https://www.reddit.com/r/dataengineering/comments/15eimaj/change_data_capture_is_still_an_antipattern_and/)
    
* [Kafka CDC blog](https://www.confluent.io/learn/change-data-capture/)