# Inside Capacitor, BigQuery’s next-generation columnar storage format

April 26, 2016

https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format

Capacitor is part of BigQuery's [BigQuery under the hood](https://cloud.google.com/blog/products/bigquery/bigquery-under-the-hood) subsystem, the storage format.

## Nested columnar representation for semistructured data

BigQuery supports several input formats — CSV, JSON, Datastore backups, AVRO. They are converted to internal nested columnar representation for semistructured data — nested and repeated fields. Parquet adopts in open source. 2010 Dremel paper.

In 2014, Google published another paper — [Storing and Querying tree-structured records in Dremel](http://research.google.com/pubs/pub43119.html) — which lays theoretical foundation and proves correctness of algorithms for filtering and aggregation operators, which take advantage of the above encoding.

## Lightweight encoding

[Integrating Compression and Execution in Column-Oriented Database Systems](https://15721.courses.cs.cmu.edu/spring2016/papers/abadi-sigmod2006.pdf) describes various techniques and encodings, such as Run Length Encoding (RLE), Dictionary encoding, Bit-Vector encoding, Frame of Reference encoding, etc.

*Example of Run Length Encoding (left) and Dictionary Encoding (right). Taken from VLDB 2009 tutorial on Column Oriented Database Systems.*

## Reordering Rows

[Reordering Rows for Better Compression: Beyond the Lexicographic Order](https://arxiv.org/abs/1207.2189)  

state of the art reordering of input rows. permutation to produce the optimial sorting of rows is a NP-complete problem. Some columns are more likely to be selected in queries than others, and some columns are more likely to be used as filters, for example in WHERE clauses. Capacitor builds an approximation model that takes into account all relevant factors and comes up with a reasonable solution. 

## BigQuery collect various statistics about the data

## Cloud Storage for shards and replication

Colossus, proper number of shards.

Colossus is a reliable and fault tolerant file system, leveraging techniques such as Reed Solomon codes to ensure that data is never lost. 

Geo-replication process, mirroring all the data into different data centers around the specified jurisdiction

## Background optimization

Background processes that constantly look at all the stored data and check if it can be optimized even further. Data compaction, encoding adjustment. Capacitor models got more trained and tuned, and it possible to enhance existing data. Atomically replaces old storage data — without interfering with running queries. Old data will be garbage-collected later.