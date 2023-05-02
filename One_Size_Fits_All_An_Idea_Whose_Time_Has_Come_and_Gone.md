# “One Size Fits All”: An Idea Whose Time Has Come and Gone

https://cs.brown.edu/~ugur/fits_all.pdf

By Michael Stonebraker, publish year 2005. This paper describes notion that the “one size fits all” theme in data bases is unlikely to succeed, the fragmentation of the DBMS market,  fracture into a collection of independent database engines is much-needed. Can't help but thinking is HTAP a false proposition?

First look back on history: System R and INGRES -> DB2 and INGRES commercial counterparts -> other vendors (Sybase, Oracle, and Informix) all use relational tables row-by-row, use B-trees for indexing, use a cost-based optimizer, and provides ACID transaction properties.

In the early 1990’s, a new trend appeared, TP and AP feature diff: BI users onto the same systems, fearing that the **complex ad-hoc queries** will degrade response time for the on-line community. So, periodically “scraped” the data from operational systems into warehouse DBMS which is bit-map indexes, materialized views, star schemas and optimizer tactics for star schema queries.

Next, heavily present a Stream processing concept which becomes trend in 5-10 years later as Flink becomes popular. It shows  discussion indicating a collection of architectural issues that result in significant differences in performance between specialized stream processing engines and traditional DBMSs. 

- Two orders of magnitude difference in observed performance between StreamBase stream processing engine (SPE).
- DBMS “Inbound” vs. Streaming system “outbound” processing. “process- after-store” is at the heart of all conventional DBMSs,

![](/Users/hlc/IdeaProjects/papers/images/One_Size_Fits_All_An_Idea_Whose_Time_Has_Come_and_Gone_1.png)

- Stream system has its own correct primitives, utilizing time windows, handle late, out-of-order, or missing messages. Allow user to specify timeout parameter, windows to be closed, saving some state (like Flink state store), StreamBase embeds BerkeleyDB for state storage

Question "One size fits all"? Substantial number of domain-specific database engines with differing capabilities off into the future. Show some specialized database engines: Data warehouses, Sensor networks, Text search, Scientific databases and XML databases.

For OLAP, it describes using this “row-store” architecture. OLTP “write-optimized” vs. “column-store”  OLAP “read- optimized” workload.



