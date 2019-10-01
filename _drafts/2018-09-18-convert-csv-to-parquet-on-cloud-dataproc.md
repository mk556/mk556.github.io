---
tags:
- dataproc
- hive
- csv
- parquet
title: Convert CSV to Parquet using Hive on Cloud Dataproc
date: 2018-09-18T03:30:00.000+00:00
description: Convert CSV to Parquet using Hive external tables on Cloud Dataproc
category: How-to
multiple_images: []

---
![](/img/0*gSryApDZXvZP0r0L.jpg)

We were recently working with a leading international voice carrier firm headquartered in US, which wanted to build a Data Warehouse on Google BigQuery.

We were provided with 40 gzipped CSV files with most of them sized \~25GB and few of them around 110GB.

![](/img/1*tTOCwoAnnnuJ3gzI4Na8JA.png)

A snapshot of the CSV files. The catch, here, was the maximum compressed CSV file that can be loaded to BigQuery (even from Cloud Storage) is [4GB](https://cloud.google.com/bigquery/quotas#load_jobs). So we were left with two alternatives :

1. Either spin up few huge VMs and unzip the files (and render all the days-worth-of-zipping obsolete ) and store it back to Google Cloud Storage.
2. Or convert the files to Parquet format.

Option 1 is a costlier and laborious way-through. Unzipping data this volume requires lot of compute power (which means money), you have to figure out using multiple cores to speed up unzip, deal with possible interruptions and resume etc.

On the contrary, Apache Parquet is columnar storage file format and designed to bring efficient columnar storage of data compared to row based files like CSV. Hive supports creating external tables pointing to gzipped files and its relatively easy to convert these external tables to Parquet and load it to Google Cloud Storage bucket. This post explains the steps using a test dataset.

Here we go —

1. Create an external table in Hive pointing to your existing zipped CSV file.
2. Create another Hive table in parquet format
3. Insert overwrite parquet table with Hive table
4. Put all the above queries in a script and submit as a job

Let’s get our hands dirty!

1. Make a folder in your bucket named ‘bike-sharing-test-data’.
2. [This](https://www.dropbox.com/s/askabysvotqijip/metro-bike-share-trip-data.csv.gz?dl=0) is a public link of a small dataset that we will use for this tutorial. Go to the link and click on _Download>Direct Download_ button in the upper right corner of the screen.
3. Upload the file to the recently created folder. The final URI would be gs://<YOUR-BUCKET>/bike-sharing-test-data/metro-bike-share-trip-data.csv.gz

The script that we will deploy is shown below:

Now, we need to upload the script to GCS :

1. Copy the script into a SQL file and replace the bucket name(manan-testing) in ‘location’ with your bucket’s name at two places in the script.
2. Upload the script anywhere outside the folder bike-sharing-test-data.

Now we have our data and script ready, let’s spin a cluster.

We will provision a Dataproc cluster consisting of preemptible and non-preemptible VMs with scheduled deletion feature.

_When and How to use preemptible VMs in Dataproc?_

1. One can use preemptible instances to lower per-hour compute costs or to create very large clusters at a lower total cost.
2. All preemptible instances added to a cluster use the [machine type](https://cloud.google.com/compute/docs/machine-types) of the cluster’s non-preemptible worker nodes.

What will happen to the stored data if my preemptible VMs are reclaimed back?

Don’t worry. Preemptibles added to a Cloud Dataproc cluster only function as processing nodes, and does not store any data( The disk space is used for local caching of data and is not available through HDFS)

_Regarding scheduled deletion property_, we cannot spin up clusters with scheduled deletion property via web UI(as of Sep’18 ).Hence, we will use gcloud command line tool to accomplish this. Furthermore, you can configure the scheduled deletion based on 3 parameters namely:

1. Maximum cluster idle time
2. At a specific Timestamp
3. Maximum age of the cluster (if 2 days, it will auto-delete 2 days after the cluster was created)

When we were processing relatively bigger files for our use case, we realised that conversion to parquet is a RAM intensive process. Hence, we chose n1-highmem-4 as the worker machine type for our use case. But for the sake of this tutorial we can set n1-standard-1 as the worker’s as well as master’s machine type. We have configured our cluster to have maximum idle time to be 30 min following which it will self destruct. You can read more about scheduled deletion [here](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/scheduled-deletion).

Fire up Cloud shell and run the following command to launch a Dataproc cluster with scheduled deletion property:

You should have Dataproc cluster up and running (You can go to Big Data > Dataproc > Clusters to check the status)

_Submitting a job via Web UI:_

1. Navigate to the script you stored in the bucket.
2. Click the more options button (three vertical dots) associated with the script.
3. Click Copy and copy the line under Source.

![](/img/1*WhLPBBlUnx5Jli1JKsgvTQ.png)Then, head over to Big Data -> Dataproc -> Jobs and click on Submit Job.

1. Set Job ID and select Region as us-central1
2. Set Cluster as ‘csv-parq-hive’
3. Set Job type as Hive
4. Select Query Source type as Query file and paste the location of the file along with the prefix “gs://” in the textbox under Query file.It’ll look similar to the following screenshot:

![Submit job page. Click Submit](/img/1*RL32BXZdouVi0DGYMY7Qfw.png)

You can click on the job name for the details. A successful run would look something like this.

![Job details](/img/0*nOgF_h2luDgZM-Fw.png)

After the job is completed, you can flicker back to your bucket to see the output. Click Refresh Bucket and head over to ‘gs://<YOUR_BUCKET_NAME>/bike-sharing-test-data/par_bike_sharing_test_data’. It’ll look something like this :

![](/img/1*8ZzBdHhhUP1JOt-AzE790w.png)
Now we’ll head towards the final ‘cherry-on-top’ step and load the parquet file into Google Bigquery:

1. Go to Bigquery and click Create Table.
2. Select Google cloud storage as source and paste the location of the parquet file generated (‘.../00000_0’).
3. Select Parquet as File Format.
4. Give the table name under Destination Table and click Create Table.

You don’t need to specify the schema when loading Parquet file because it is a self-describing data format which embeds the schema, or structure, within the data itself.

A huge shout-out to [RK Kuppala](https://medium.com/u/5866d707938) for providing the crucial guidance and support along the way.

Voila! Hope this post helped you :)