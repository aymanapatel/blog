---
title: "ABCs of Databases"
seoTitle: "ABCs of Databases"
seoDescription: "Understanding the ABCs of Databases; ACID, BASE and CAP"
datePublished: Sun Jan 14 2024 16:39:17 GMT+0000 (Coordinated Universal Time)
cuid: clrdq18xe000309l70dicg13h
slug: abcs-of-databases
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705249300114/52941de8-909d-40d4-ae0b-a4c1c839877b.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1705250314073/3efb035b-f59f-44c8-a9f3-22571ab8f80d.png
tags: postgresql, nosql, databases, sql, cap-theorem

---

When we talk about creating APIs with a database for storage; we always think database to be a storage layer which would do its job of doing CRUD operations. But their is a unconscious belief of the API to be single user, single transaction. Even if we think that multiple simultaneous read/writes happen; we assume that data will be in the state that we want. But there are a lot of nuances in how data gets persisted. In order to know about this; there are several topics that need to be covered. These could be:

1. ABC of database:
    
    1. ACID: How the database implements ACID. Is it a NoSQL (Mongo, Redis, Cassandra, HBase) database? If so, how does it handle mutli-writer same- time problem?
        
    2. BASE: For NoSQL, what are the tradeoffs of using BASE? How does NoSQL implements Eventual consistency under the hood?
        
    3. CAP Theorem: Which one of AP from CAP theorem does it (for NoSQL) does it support? Is CAP even a good acronym for understanding different types of database systems?
        
2. What **Transaction Isolation Levels** is configured at database? What is impact of consistency vs latency with these settings?
    
3. **Queries planner and Execution**: How does your query formed by your database affecting the performance of the system
    

All these are just high level questions which need to be seen in depth. The first point (i.e ABC of Database) is the starting point to understand the rest of the points; which will be the focus of this blog.

# ABCs of Database

Just putting the acronym out there:

| Acronym | 1st | 2nd | 3rd | 4th |
| --- | --- | --- | --- | --- |
| ACID | Atomicity | Consistency | Isolation | Durability |
| BASE | Basic ... | ... Availability | Soft State | Eventual Consistency |
| CAP | Consistency | Availability | Partition-tolerance |  |

# ACID

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249498944/2c94a09d-9982-40ef-99e3-c40b78f740c9.jpeg align="center")

## 1.a. Atomicity

It states that transaction has started; it should either be completed or rolled-back if an error occurs. In DB terms; either transaction should be `COMMITTED` or `ABORTED` This mechanism is achieved by

1. **REDO/UNDO mechanisms** such as REDO/UNDO logs to bring data to the correct atomic state etc
    
2. **Shadow Paging** It follows **Copy-on-write** mechanism where the parent process forks and creates a **shadow page** for *uncommitted transaction* which is either: F
    
    1. If txn is successful, Shadow Page is flushed to original page/disk OR
        
    2. If txn is erroneous; Shadow page is discarded as original page has consistent committed data.
        

> **All in or Nothing**

## 1.b. Consistency

This rule is little bit confusing. Especially when you pair up with BASE(Eventual consistency) and CAP. In spite of confusion; Consistency from ACID is the clear and most thought out definition out of the 3 (ACID, BASE and CAP) acronyms. It states that once database starts the transaction consistently, it should end consistently. Consistency is enforced by Applications with the Integrity Constraints.

> Transaction starts in Consistent Manner; And Ends in Consistent Manner(All Integrity constraints are followed)

Consistency is enforced by **integrity constraints**. This could be Primary Key, or Constraints such as Foreign Key, `NULL` constratints, `CHECK` value constraints such as `ACCOUT>=100` etc.

## 1.c. Isolation

It states the transactions should be isolated from each other.

For example; Given Bank Account has *100$* , When Alice has a card that they withdraw *25$* and Bob starts the transaction simultaneously and withdraws *30$* Bank account at the end is 100-(25+30) = *45$*

> The Math checks out.

## 1.d. Durability

It states that if transaction has started and completed (`COMMIT` is done); its effect should be persisted, even if there is a system failure. If there was a transaction that has not been completed, it should be rolled back to the previous completed state. It should be noted that databases commit first to Buffer Pool and then to disk. Durability is a guarantee at the disk level.  
Similarly to Atomicity; Durability uses `REDO/UNDO` or **Shadow Paging** mechanism to ensure Durable state.

> Are you durable, even when you are down and out?

# BASE

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249530139/5be0e449-c8ff-4340-9e6b-13641b1a0f7c.jpeg align="center")

Before we go into BASE; we need to know the history of misnomers in database. BASE and NoSQL are really what they are intended to be. Just for sake of blog, I have added this section. This acronym (BASE) should be taken by grain of salt. This is marketing gimmick in order to hide the real important concepts such as CAP (and later PACLEC) and the OG ACID.

Even the acronym is the mash of 3 things (Basic Availability, Soft state and Eventual Consistency) fitting into a 4 letter acronym and an alternative to ACID from the chemistry world.

## 2.a. Basic Availability

As NoSQL prioritises Scalability and Availability over transaction correctness; it needs to be available at all times with highest five 9s percentile (99.999).

## 2.b. Soft State

This is related to eventual consistency. It basically is a disclaimer that the data available in the database is not the final state. Due to eventual consistency across various nodes; the data will not be guaranteed to be *write-consistent* or *mutually consistent* across nodes.

## 2.c. Eventual Consistency

As there are multiple machines in NoSQL databases due to various reasons such as sharding, horizontal scaling and goal is to be as fast as possible; data might be distributed across various nodes; and whichever node gives the answer first; is treated as the response to the API request. This means that data is consistent eventually across multiple machines. In the initial days, consensus was not the norm in NoSQL databases and hack ways to fan-out requests and pick the first one in order to be first was the norm. Consensus protocol such as Raft and Paxos where then integrated to have consistency (at the expense of performance.)

# CAP

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249570779/bfc0cf9b-f587-48e0-9ee8-2129ff306ccd.jpeg align="center")

## 3.a. Consistency

This is not to be confused with Consistency in ACID.

C in ACID is meant for transaction as a series of steps inside the single node; whereas Consistency in CAP theorem considers a distributed environment where there are 2 or more machines/nodes. It resembles more of **linearizability**

> Formal definition of linearizability: If operation B started after operation A successfully completed, then operation B must see the the system in the same state as it was on completion of operation A, or a newer state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249583483/077ea9dd-8025-4bb0-be6a-2e57a3d34d95.png align="center")

In the diagram, there are 5 steps which ensure data is distributed consistently.

1. *Server A* sets A's value as **3** to the *primary database*
    
2. *Primary database* starts to propagate this info to the *Replica database*
    
3. `A=3` travels across the network and reaches
    
4. *Primary database* acknoledges that this info has be sent to *Replica database*
    
5. *Server B* reads from *Replica database* and gets `A=3`
    

if step #4 .i.e. Primary Acknowledging that transaction has been **committed**, then the replicas would have consistent data immediately (No Eventual consistency)

Consistency in CAP in simple terms:

> Alway return **up-to-date** information

## 3.b. Availability

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249589564/846f1822-6dcb-4c83-a7d5-fa2aa9a7275a.png align="center")

Consider that one of databases is down and consider the diagram:

1. *Server A* sets A's value as **3** to the *primary database*
    
2. *Primary database* starts to propagate this info to the *Replica database*; but finds that *Replcia database* is down
    
3. *Server B* reads from *Primary database* and gets `A=3`
    

Availability in CAP in simple terms:

> System **must** return information, even if out-of-date/stale. Even if node goes down, there will be another node that brings information.

## 3.c. Partition Tolerance

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249602401/bad3d541-b9b0-4473-9abc-7434c9288bc2.png align="center")

In the diagram above; the network is down. In this case Server A and Server B are on their own. So what do they do? They read from their own databases. *Server B's* database thinks of itself as the primary database and it cannot connect to the primary database to get the latest info. When the network is established, the database have a reconciliation process wherein the data is brought to a "consistent". Note the asterisks; the consistent can become messy. Some databases look into lamport clocks to figure to the last update. If things are even more messy, then the application code does some woodo magic to make the data consistent enough.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249609356/26c5d9ca-03f8-4741-a816-dfaa9bcf42f2.png align="center")

Partition Network tolerance in CAP in simple terms:

> System continue operating even if **Network link has been severed**.

CAP theorem states that a distributed system can follow only 2 out of 3 acronyms. The trade-off is always there when you have more than a single database. All in all, there is no free lunch.

### CA

CA means that data is the latest COMMIT which meands there are no inconsistent data in the distributed data sources as a whole. Mostly relational databases follow this.

### CP

Theses systems will not always answer if they are not **available**. But if they do, you can be sure that the answer is correct due to its **consistency guarantees**.

### AP

These systems will always give an answer, even if it is not the latest. Social media sites in early days; in hope to be very highly available and requiring partition for different regions/campuses (like Facebook), had loads of bugs where site loaded but posts and comments would come and go.

The following VEN diagram is the list of databases supporting CA, CP and AP schemes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249617984/f8ba5a7a-0ea4-4e07-983e-c5c9712a9367.png align="left")

### Critique of CAP theorem

In spite of bringing a way of trade-off analysis to distributed system, in real world, it is not sufficient model to understand tradeoffs. A lot of times, Partition tolerance is needed and not a optional thing in the age of Mult-node deployments and on-soil regulations. Other critique is the network failure as the only failure is a wrong assumption. Databases also suffers from Murphy;s law. Power outage, Disk corruption etc are not part of CAP but are real scenarios that need to be considered. Martin Klepmann has a good blog on the critique of the CAP theorem which can be found [here](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html). (Paper format with even more details are [here](https://arxiv.org/pdf/1509.05393.pdf))

# PACLEC: A better CAP

> A more nuanced framework for comparing NoSQL databases

It stands for: Partition Tolerant Always Available Consistent Else, choose Latency Consistency

And yes, it is an acronym with a `IF/ELSE` statement.

* If (P)artition; Then (A)vailability and (C)onsistency
    
* (E)lse; (L)atency and (C)onsistency
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705249666664/d3d4bf51-c111-4e31-8aca-b19a55eac6d9.png align="left")

* Let us consider the left-hand side. If partition is all good; then you follow CAP theoem; albeit the CP or AP rule only.
    
* For the right-hand side, if network partition is down; then you have to make a tradeoff between Latency and Consistency. If you want your system to be fast then you have to sacrifice consistent data. Conversely, if you want your data to be consistent, then you have to wait because of internal consensus (RAFT, Paxos, whatever) of the correct data.
    

| Database | P+A | P+C | E+L | E+C |
| --- | --- | --- | --- | --- |
| BigTable |  | ✅ |  | ✅ |
| HBASE |  | ✅ |  | ✅ |
| Mongo | ✅ |  |  | ✅ |
| MySQL/Postgress (and other RDBMS) |  | ✅ |  | ✅ |
| Cassandra | ✅ |  | ✅ |  |
| Scylla (Cassandra fork) | ✅ |  | ✅ |  |
| Google Spanner |  | ✅ |  | ✅ |
| CockroachDB |  | ✅ |  | ✅ |

Watch this video from Dr Daniel Abadi on PACLEC (author of the concept) from [ScyllaDB channel](https://youtu.be/vnXXFpySYVE)

# Closing thoughts

This is just the start of understanding the basic terms of databases. There are many more concepts such as Memory Management, Buffer Cache, DB data-structures, Query planning & Execution, 2PC & Quorum, MVCC, Columnar Storage, WAL & Journaling and much more which is greatly covered in these 2 courses

1\. CMU 15-721: Advanced database systems: [Playlist](https://www.youtube.com/watch?v=uikbtpVZS2s&list=PLSE8ODhjZXjaKScG3l0nuOiDTTqpfnWFf)

2\. CMU 445-645: Advanced database systems [Playlist](https://www.youtube.com/watch?v=LWS8LEQAUVc&list=PLSE8ODhjZXjYzlLMbX3cR0sxWnRM7CLFn)