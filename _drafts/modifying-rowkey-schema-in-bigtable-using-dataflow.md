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
- "/uploads/max-workers-dataflow-template.jpg"
- "/uploads/changing-rowkey-bigtable.jpeg"
- "/uploads/Create Dataflow job from template - searce-sandbox - Google Cloud Platform
  2019-10-07 13-33-21.jpg"
image: "/uploads/will-b-UXKNbZjHCyw-unsplash.jpg"
category: bigtable

---
image caption - Photo by [Will B](https://unsplash.com/@willbro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/wide?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

Cloud Bigtable is a petabyte-scale, fully managed NoSQL database service in GCP for large analytical and operational workloads. It supports the open source industry standard [HBase API](https://hbase.apache.org/), and has integrations with GraphDBs, TSDBs, Geospatial DBs ( [link](https://cloud.google.com/bigtable/docs/integrations) ). Actually, Bigtable was initially released in 2005, but wasn't available to general public until 2015. Apache HBase was created based on Google's publication [Bigtable: A Distributed Storage System for Structured Data](http://research.google.com/archive/bigtable.html) with initial release in 2008.

Bigtable has only one primary index known as rowkey which can look like 'Field1#Field2#Field3', if you decide to have multi-value rowkey. A rowkey should be designed with keeping future queries in mind. I'll not go deep into things to keep in mind while designing the rowkey - you can find it [here](https://cloud.google.com/bigtable/docs/schema-design) and [designing rowkey for time series data](https://cloud.google.com/bigtable/docs/schema-design-time-series).

There can be several instances where you need to modify your rowkey - first being load testing several rowkeys and figuring out the best one for you. Also, A table with particular rowkey can serve only one type of query better, you might want to store the same data with different rowkey for another type of queries to perform efficiently.

That being said, let's see how we will achieve this -

1. Exporting Bigtable data to GCS in Avro format by launching this open source Cloud Dataflow job ([Bigtable-to-GCS-avro](https://cloud.google.com/dataflow/docs/guides/templates/provided-batch#cloudbigtabletoavrofile)).
2. Creating a empty table in Bigtable which will contain rows with updated rowkey (Alternatively, you can create a new Bigtable cluster as well, if you don't want it to affect your existing Bigtable).
3. Importing Bigtable's rows dump from GCS to new empty table after modifying a little code of [GCS to Bigtable template](https://cloud.google.com/dataflow/docs/guides/templates/provided-batch#cloud-storage-avro-to-cloud-bigtable) and launching this as Cloud Dataflow job.

Let's start -

1. We can launch the export job (Bigtable to GCS) directly from GCP console. Go to **Big data -> Dataflow -> Create Job from template.** You can fill out the details as shown below -

   ![](/uploads/Create Dataflow job from template - searce-sandbox - Google Cloud Platform 2019-10-07 13-33-21.jpg)

You can set the max-workers property to 10 and and instance type to n1-standard-4.

> I recommend using minimum n1-standard-2 as I faced OutOfMemory errors while using n1-standard-1 workers in Dataflow job.

> Also, number of vCPUs (workers * vCPU per machine) should be proportional to number of Bigtable nodes else it would lead to either over-utilization (Too many workers reading from small Bigtable cluster ) or under-utilization (small number of workers reading from relatively larger Bigtable cluster - resulting in more execution time).

For my use case, I had to take dump of 500GB table in Bigtable. I increased Bigtable nodes to 12 and instance-type = n1-standard-4 and max-workers to 10. The export job took around \~45 mins. 

> Note : If you don't set max-workers to any number. Dataflow can scale exponentially to 600 or 700 VMs based on size of your table. So, it's good practice to have an upper bound on max VMs

## References

1. [https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable](https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable "https://stackoverflow.com/questions/24860516/what-it-the-difference-between-hbase-and-bigtable")
2. 