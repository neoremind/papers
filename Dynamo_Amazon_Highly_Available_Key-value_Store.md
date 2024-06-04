# Dynamo: Amazon’s Highly Available Key-value Store

## ABSTRACT
Reliability at massive scale is one of the biggest challenges we face at Amazon.com, one of the largest e-commerce operations in the world; even the slightest outage has significant financial consequences and impacts customer trust. The Amazon.com platform, which provides services for many web sites worldwide, is implemented on top of an infrastructure of tens of thousands of servers and network components located in many datacenters around the world. At this scale, small and large components fail continuously and the way persistent state is managed in the face of these failures drives the reliability and scalability of the software systems.

This paper presents the design and implementation of Dynamo, a highly available key-value storage system that some of Amazon’s core services use to provide an “always-on” experience. To achieve this level of availability, Dynamo sacrifices consistency under certain failure scenarios. It makes extensive use of object versioning and application-assisted conflict resolution in a manner that provides a novel interface for developers to use.

## 1. INTRODUCTION

### Reliability and Scalability is the topmost

Amazon has strict operational requirements of reliability and scalability (performance and efficiency also matters). System needs to treat failure handling as the normal case.

### What is DynamoDB?

Dynamo is highly available and scalable distributed data store. 

It is an eventually-consistent storage system. AP system in CAP theory. 

Dynamo is used to manage the state of services that have very high reliability requirements and need tight control over the tradeoffs between availability, consistency, cost-effectiveness and performance. 

Dynamo is a primary key based simple key/value (binary objects, blobs) system.

### How 
- Data is partitioned and replicated using consistent hashing. 
- Consistency is facilitated by object versioning. 
- The consistency among replicas during updates is maintained by a quorum-like technique and a decentralized replica synchronization protocol. 
- Gossip based distributed failure detection and membership protocol. 
- Decentralized system with minimal need for manual administration. 


### Assumptions and Requirements

- Query Model: No relational.
- ACID Properties: weaker consistency for high availability. no isolation guarantees. only single key updates.
- Efficiency and SLA: p99.9 latency is meaningful and stringent. In implementation, 99.9th percentile latencies are around 200 ms and are an order of magnitude higher than the averages.

### Design Considerations (Tenets of the storage system)

1. **Optimistic replication techniques and eventually consistent** vs. synchronous replica coordination and strong consistent . So, when and who to resolve update conflicts? During read by the client since Dynamo is “always writeable” data store and “merge” the conflicting versions by client is suitable.

2. Others: **Incremental scalability, Symmetry, Decentralization, Heterogeneity**.


zero-hop DHT

## 2. SYSTEM ARCHITECTURE

![](/images/Dynamo_Amazon_Highly_Available_Key-value_Store/2.png)

### System Interface

- **get(key)**: returns a single object or a list of objects with conflicting versions along with a context. 
- **put(key, context, object)**

### Partitioning and Replication

Applies a MD5 hash on the key to generate a 128-bit identifier to do routing. Use **consistent hashing** to distribute the load across multiple storage hosts. To avoid non-uniform data and load distribution, use **“virtual nodes”**.

Replicates these keys at the N-1 clockwise successor nodes in the ring. (N=3 in the example)

![](/images/Dynamo_Amazon_Highly_Available_Key-value_Store/1.png)

**Coordinator** receives request from Load Balancer, if the node is not in the top N of the requested key’s preference list, that node will forward the request.

**R + W > N** a quorum-like system, client tunes NWR to achieve their desired levels of performance, availability and durability. Common use: (N,R,W) is (3,2,2). Example clients are shown in section 6. EXPERIENCES & LESSONS LEARNED.

### Data Versioning

No up-to-date read is possible. Under certain failure scenarios (e.g., server outages or network partitions), updates may not arrive at all replicas for an extended period of time.

Dynamo treats each updsate as a new and immutable **version** of the data. Client performs update must has a version captured in context.

Version branching may happen, client must perform the reconciliation in order to "merge" multiple branches of data evolution back into one.

Dynamo uses **vector clocks** [12] in order to capture **causality** between different versions of the same object. Fix "last write win" leading to lost update, preserve write latency and availability, the system is sacrificing consistency requirements. See more on *Time, Clocks, and the Ordering of Events in a Distributed System.* from Leslie Lamport. 

![](/images/Dynamo_Amazon_Highly_Available_Key-value_Store/3.png)

Side notes: Not Dynamo internally does not reliant on vector clock instead uses synchronous replication.

### Handling Failures

- **Sloppy Quorum and Hinted Handoff**: ensures that the read and write operations are not failed due to temporary node or network failures. In the example Ring, if A is temporarily down or unreachable during write, then write will be sent to node D. Upon detecting that A has recovered, D will attempt to deliver the replica back to A. 

- **Anti-entropy with Merkle Trees**: Merkle Trees are used in Dynamo to identify any inconsistencies between nodes. Individual nodes maintain separate Merkle Trees for the key ranges that they store and they can be used to quickly work out if data is consistent and if not, identify which pieces of information need to be synchronised.

![](/images/Dynamo_Amazon_Highly_Available_Key-value_Store/4.png)

*see https://blog.scottlogic.com/2020/09/08/deconstructing-dynamo.html*

### Cluster Membership

Dynamo is a decentralized, distributed system; there is no central hub where information about configuration or the topology of the cluster can be accessed. 

A **gossip-based protocol** propagates membership changes and maintains an eventually consistent view of membership.

## 3. IMPLEMENTATION

Storage node consists of 3 components: request coordination, membership and failure detection, and a local persistence engine. Implemented in Java.

Berkeley Database (BDB) Transactional Data Store.



