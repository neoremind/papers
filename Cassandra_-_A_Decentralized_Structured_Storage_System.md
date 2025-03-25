# Cassandra - A Decentralized Structured Storage System

2009 https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf

## ABSTRACT

Cassandra is a distributed storage system for managing very
large amounts of structured data spread out across many
commodity servers, while providing highly available service
with no single point of failure. Cassandra aims to run on top
of an infrastructure of hundreds of nodes (possibly spread
across diﬀerent data centers). At this scale, small and large
components fail continuously. The way Cassandra man-
ages the persistent state in the face of these failures drives
the reliability and scalability of the software systems rely-
ing on this service. While in many ways Cassandra resem-
bles a database and shares many design and implementation
strategies therewith, Cassandra does not support a full rela-
tional data model; instead, it provides clients with a simple
data model that supports dynamic control over data lay-
out and format. Cassandra system was designed to run on
cheap commodity hardware and handle high write through-
put while not sacrificing read eﬃciency.

- from Facebook supporting Inbox Search in 2008
- treats failures as the norm 
- meet scalability (P) and availability (A) in CAP. Eventual Consistency of writes to a single table

**XU: Cassandra is a distributed/highly scalable storage system partitioned by consistent hashing with gossip and each node implements LSM-tree local engine, it allows multi-master replication using versioned data and tunable consistency**

## 1. DATA MODEL & API

- table is distributed multi dimensional map indexed by a key.
- row key level atomicity 
- value ::= highly structured object
- column family like BigTable
- sort order by time or by name

```
insert(table,key,rowMutation)
get(table,key,columnName)
delete(table,key,columnName)
```

## 2. SYSTEM ARCHITECTURE

Requirement:
- scalable and robust solutions for load balancing
- membership and failure detection,
- failure recovery
- replica synchronization
- overload handling,
- state transfer
- concurrency and job scheduling, 
- request marshalling, 
- request routing, 
- system monitoring and alarming,
- configuration management

Typically a read/write request for a key gets routed to any node in the Cassandra cluster. 

The node then determines the replicas for this particular key. 

For writes, the system routes the requests to the replicas and waits for a quorum of replicas to acknowledge the completion of the writes. 

For reads, based on the consistency guarantees required by the client, the system either routes the requests to the closest replica or routes the requests to all replicas and waits for a quorum of responses.

### 2.1 Partitioning

consistent hashing w/ an order preserving hash function - dynamically partition the data over the set of nodes.

How: walking the ring clockwise to find the first node with a position larger than the item’s position. 

Benefit: only aﬀects its immediate neighbors and other nodes remain unaﬀected.

Optimization: Multiple Tokens per Physical Node (vnodes), see https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html

To address non-uniform data and load distribution, analyze load information on the ring and have lightly loaded nodes move on the ring. *[Xu: make certain row key range more dense]*

```
CREATE TABLE t (
   id int,
   k int,
   v text,
   PRIMARY KEY (id)
);
```

`id` is used to generate the partition key and the second field `c` is the clustering key used for sorting within a partition.

### 2.2 Replication

Each data item is replicated at N hosts, where N is the replication factor configured “per-instance”

Replication policies:
- Rack Unaware
- Rack Aware (within a datacenter)
- Datacenter Aware

every node is aware of every other node in the system and hence the range they are responsible for.

### 2.3 Membership

Anti-entropy Gossip called Scuttlebutt from paper *Eﬃcient reconciliation and
flow control for anti-entropy protocols.*

Φ Accrual Failure Detector //TODO

### 2.4 Bootstrapping

### 2.5 Scaling the Cluster

node splitting

### 2.6 Local Persistence

Log Structured Merge Tree (LSM)

- WAL/commit log: journaled, a write into a commit log for durability
and recoverability, sequential commit log write and maximize disk throughput. 

- Memtables: an update into an in-memory data
structure after writing into the commit log. 

- dump memtable to SSTables like leveldb.

- index for eﬃcient lookup based
on row key. indices are also persisted along with the
data file.

- merge process runs in the background to compact.

- bloom filter on the key to quickly prune files that may overlap in keys.

### 2.7 Implementation Details

3 Modules
- partitioning module
- cluster membership and failure detection module 
- storage engine module

SEDA and Java.

fast sync mode the writes to the commit log are buﬀered in page cache.

lockless for read/write operations

## Appendix

https://cassandra.apache.org/doc/latest/cassandra/architecture/overview.html

Combination of Amazon’s Dynamo distributed storage and replication techniques and Google’s Bigtable data and storage engine model.

Objective:

- Full multi-primary database replication
- Global availability at low latency
- Scaling out on commodity hardware
- Linear throughput increase with each additional processor
- Online load balancing and cluster growth
- Partitioned key-oriented queries
- Flexible schema

### Cassandra Query Language (CQL)

### ScyllaDB

- close-to-the-metal architecture handles millions of ops/sec with predictable single-digit millisecond latencies. - 
- API-compatible with Apache Cassandra and Amazon DynamoDB. 
- shared-nothing approach that increases throughput and storage capacity to realize order-of-magnitude performance improvements and reduce hardware costs.
