# Kudu: Storage for Fast Analytics on Fast Data

## Abstract

Kudu is an open source storage engine for structured data which supports low-latency random access together with ef- ficient analytical access patterns. Kudu distributes data us- ing horizontal partitioning and replicates each partition us- ing Raft consensus, providing low mean-time-to-recovery and low tail latencies. Kudu is designed within the context of the Hadoop ecosystem and supports many modes of access via tools such as Cloudera Impala[20], Apache Spark[28], and MapReduce[17].

## Introduction

Kudu is a new storage system designed  to fill this gap between high-throughput sequential-access storage systems such as Parquet, HDFS and low- latency random-access systems such as HBase or Cassandra.

![gswk_0108](/Users/zhangxuv/Downloads/Kudu_Storage_for_Fast_Analytics_on_Fast_Data_1.png)from https://www.oreilly.com/library/view/getting-started-with/9781491980248/ch01.html

## Kudu at a high level

Kudo is for structured data with schema, providing Insert, Update, and Delete APIs.

```
CREATE TABLE my_first_table
(
  id BIGINT,
  name STRING,
  PRIMARY KEY(id)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES ('kudu.num_tablet_replicas' = 'n[3 or 5]');

// PARTITION BY RANGE
```

Kudo's Consistency Model are Snapshot Consistency, with that read a consistent snapshot. No external consistency guarantee, but offers the option to manually propagate timestamps between clients. Kudu optionally uses commit-wait as in Spanner.

## Architecture

![Kudu_Storage_for_Fast_Analytics_on_Fast_Data_2](images/Kudu_Storage_for_Fast_Analytics_on_Fast_Data_2.png)

from https://kudu.apache.org/docs/

Partitioning is 10-100 tablets per machine. Each tablet can be tens of GB. Hash-partitioning or range-partitioning.

Replication uses Raft consensus algorithm. Kudo did some refinement on Raft, please refer to #3.3.

Kudu Master is a catalog manager, a cluster coordinator and  a tablet directory.

## Tablet storage

RowSets := 1 MemRowSet (disjoint sets of rows) + Multiple DiskRowSets (may overlap in pk) in one tablet. Similar to LevelDB, but only with level 0 and 1.

A background thread periodically flushes MemRowSets to disk to form DiskRowSets.

### MemRowSet

MemRowSet is a in-memory concurrent B-tree with optimistic locking, broadly based off the design of MassTree. Optimizaion appraoches include 

* do not support removal of elements from the tree, use MVCC.
* do not support arbitrary in-place updates of records in the tree. Instead, we allow only modifications which do not change the value’s size?
* larger internal and leaf nodes sized at four cache-lines (256 bytes) each.
* utilize SSE2 mem- ory prefetch instructions to prefetch one leaf node ahead of our scanner, and JIT-compile record projection operations using LLVM.
* efficient tree traversal using only memcmp operations for comparison of pk.

### DiskRowSet

DiskRowSet := base data + delta stores (updates and deletes, like REDO log, also use UNDO log in Snapshot reading). 

Roll the DiskRowSet after each 32 MB of IO.

Similar to Parquet, Column -> Page. Encoding: dictionary encoding, bitshuffle, front coding + optionally compression (LZ4, gzip, or bzip2).

Page level index: An embedded B-tree index allows efficient seeking to each page based on its ordinal offset within the rowset.

A primary key index column, which stores the encoded primary key for each row.  chunked Bloom filter for fast pruning, may false positive.

**When servicing an update to data within a DiskRowSet, we first consult the primary key index column. By using its embedded B-tree index, we can efficiently seek to the page containing the target row. Using page-level metadata, we can determine the row offset for the first cell within that page. By searching within the page (eg via in-memory binary search) we can then calculate the target row’s offset within the entire DiskRowSet. Upon determining this offset, we insert a new delta record into the rowset’s DeltaMemStore.**

```
Column
+----+
|    | Page
+----+
|    |
+----+
|    | --> Page level index B-tree index with ordinal offset and min max metadata
+----+
|    | primary key index B-tree
+----+
|    | primary key Bloom filter 4KB page
+----+

```

Delta stores are either in-memory DeltaMemStores, or on-disk DeltaFiles. Delta stores maintain a mapping from (row offset, timestamp) tuples to RowChange-List records? The timestamp is the MVCC timestamp assigned when the operation was originally written. The RowChangeList is a binary-encoded list of changes to a row, for example indicating SET column id 3 = ‘foo’ or DELETE.

Read Path: Kudu performs the scan one column at a time. First, it seeks the target column to the correct row offset (0, if no predicate was provided, or the start row, if it previously determined a lower bound). Next, it copies cells from the source column into our row batch using the page-encoding specific decoder. Last, it consult the delta stores to see if any later updates have replaced cells with newer versions, based on the current scan’s MVCC snapshot, applying those changes to our in-memory batch as necessary. Because deltas are stored based on numerical row offsets rather than primary keys, this delta application process is extremely efficient: it does not re- quire any per-row branching or expensive string comparisons.

After performing this process for each row in the projection, it returns the batch results.

Lazy Materialization

RowSet Compaction

Scheduling maintenance: knapsack problem





