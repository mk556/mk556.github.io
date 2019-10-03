---
tags:
- bigtable
- dataflow
- avro
- gcp
title: Modifying Rowkey (Schema) in Bigtable using Dataflow
date: 2019-10-03 18:00:00 +0530
description: Changing the rowkey in Bigtable using dataflow
multiple_images:
- "/uploads/changing-rowkey-bigtable.jpeg"
image: "/uploads/will-b-UXKNbZjHCyw-unsplash.jpg"
category: bigtable

---
Cloud Bigtable is a petabyte-scale, fully managed NoSQL database service in GCP for large analytical and operational workloads. It supports the open source industry standard [HBase API](https://hbase.apache.org/), and has integrations with GraphDBs, TSDBs, Geospatial DBs ( [link](https://cloud.google.com/bigtable/docs/integrations) ). Actually, Bigtable was initially released in 2005, but wasn't available to general public until 2015. Apache HBase was created based on Google's publication [Bigtable: A Distributed Storage System for Structured Data](http://research.google.com/archive/bigtable.html) with initial release in 2008.

Bigtable has only one primary index known as Rowkey which can look like 'Field1#Field2#Field3', if you decide to have multi-value rowkey. A rowkey is designed with keeping future queries in mind. A rowkey is optimized for one type of query and cannot perform optimally for all queries. In the sample rowkey above, priority relationship is Field1 > Field2 > Field3, meaning, you can fire queries saying give me all the rows for which Field1 is 'abc' but you cannot say give me all rows with Field2 = 'abc'. Ideally, to get most out of Bigtable, you should give specific value of 

You can read more about designing a rowkey [here](https://cloud.google.com/bigtable/docs/schema-design) and things to keep in mind while [designing rowkey for time series data](https://cloud.google.com/bigtable/docs/schema-design-time-series). 

## References

1. [https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable](https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable "https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable")
2. 