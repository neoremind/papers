# KSQL: Streaming SQL Engine for Apache Kafka

## ABSTRACT

Demand for real-time stream processing has been increasing and
Apache Kafka has become the de-facto streaming data platform in
many organizations. Kafka Streams API along with several other
open source stream processing systems can be used to process
the streaming data in Kafka, however, these systems have very
high barrier of entry and require programming in languages such
as Java or Scala.
In this paper, we present KSQL, a streaming SQL engine for
Apache Kafka. KSQL provides a simple and completely interactive
SQL interface for stream processing on Apache Kafka; no need
to write code in a programming language such as Java or Python.
KSQL is open-source, distributed, scalable, reliable, and real-time.
It supports a wide range of powerful stream processing opera-
tions including aggregations, joins, windowing, sessionization,
and much more. It is extensible using User Defined Functions
(UDFs) and User Defined Aggregate Functions (UDAFs). KSQL
is implemented on Kafka Streams API which means it provides
exactly once delivery guarantee, linear scalability, fault tolerance
and can run as a library without requiring a separate cluster.