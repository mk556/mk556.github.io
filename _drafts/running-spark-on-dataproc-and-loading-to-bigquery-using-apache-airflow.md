---
tags:
- bigquery
- airflow
- dataproc
- spark
title: Running Spark on Dataproc and loading to BigQuery using Apache Airflow
date: 2019-10-01 10:00:00 +0530
description: The Airflow Script checks Google Cloud Storage for specified directory,
  creates a Dataproc cluster, runs a spark job and loads the output of Spark to Google
  Bigquery.
category: How-to
multiple_images:
- "/uploads/mahkeo-weaNmPm4TqA-unsplash.jpg"

---
Apache Airflow is an popular open-source orchestration tool having lots of connectors to popular services and all major clouds. Today we will explore a pipeline which automates the flow from incoming data to Google Cloud Storage, Dataproc cluster administration, running spark jobs and finally loading the output of spark jobs to Google BigQuery.

![](/uploads/mahkeo-weaNmPm4TqA-unsplash.jpg)

So let's get started!

You can find the entire python file [here](https://github.com/mk556/airflow-scripts/blob/master/gcs-dataproc-bigquery.py). In this blog post, I'll go through each component.

## GCS Prefix check

```python
def dynamic_date(date_offset):

    date_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%d\") }}"
    month_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%m\") }}"
    year_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%Y\") }}"

    return {"date":date_config,"month":month_config,"year":year_config}


def gcs_prefix_check(date_offset):

    date_dict = dynamic_date(date_offset)
    return date_dict["year"]+"/"+date_dict["month"]+"/"+date_dict["date"]


gcs_prefix_check = GoogleCloudStoragePrefixSensor(
    dag=dag,
    task_id="gcs_prefix_check",
    bucket="example-bucket",
    prefix="dir1/dir2"+gcs_prefix_check(3)
)
```