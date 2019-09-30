---
tags:
- bigquery
- airflow
- dataproc
- spark
title: Running Spark on Dataproc and loading to BigQuery using Apache Airflow
date: 2019-10-01T04:30:00.000+00:00
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
{% raw %}
def dynamic_date(date_offset):
    ''' subtracts date_offset from execution_date and returns a tuple'''

    date_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%d\") }}"
    month_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%m\") }}"
    year_config = "{{ (execution_date - macros.timedelta(days="+str(date_offset)+")).strftime(\"%Y\") }}"

    return {"date":date_config,"month":month_config,"year":year_config}


def gcs_prefix_check(date_offset):
    ''' returns string in format YYYY/MM/DD emulating sample directory structure in GCS'''

    date_dict = dynamic_date(date_offset)
    return date_dict["year"]+"/"+date_dict["month"]+"/"+date_dict["date"]


gcs_prefix_check = GoogleCloudStoragePrefixSensor(
    dag=dag,
    task_id="gcs_prefix_check",
    bucket="example-bucket",
    prefix="dir1/dir2"+gcs_prefix_check(3)
) # GoogleCloudStoragePrefixSensor checks GCS for the existence of any BLOB which matches operator's prefix
{% endraw %}
```

Sensors in Airflow are operators which usually wait for a certain entity or certain period of time. Few available sensors are TimeDeltaSensor, file, database row, S3 key, Hive partition etc. Our requirement was that the flow should initialize as soon as the raw data is ready in GCS (uploaded by say x provider). Here I have used GCS Prefix Sensor which makes a synchronous call to GCS and checks if there is any BLOB whose URI matches with specified prefix. If yes, the task state becomes success else it waits for certain period of time before rechecking. `dynamic_date_` _and_ `gcs_prefix_check` are helper functions which builds prefix dynamically. `dynamic_date` function can be used if there is a lag between data arrival and data processing date.

## Start and Stop Dataproc Cluster

```python 
{% raw %}
start_cluster_example = DataprocClusterCreateOperator(
        dag=dag,
        task_id='start_cluster_example',
        cluster_name='example-{{ ds }}',
        project_id= "your-project-id",
        num_workers=2,
        num_preemptible_workers=4,
        master_machine_type='n1-standard-4',
        worker_machine_type='n1-standard-4c',
        worker_disk_size=300,
        master_disk_size=300,
        image_version='1.4-debian9',
        init_actions_uris=['gs://bootscripts-bucket/bootstrap_scripts/bootstrap-gcp.sh'],
        tags=['allow-dataproc-internal'],
        region="us-central1",
        zone='us-central1-c',#Variable.get('gc_zone'),
        storage_bucket = "dataproc-example-staging",
        labels = {'product' : 'sample-label'},
        service_account_scopes = ['https://www.googleapis.com/auth/cloud-platform'],
        properties={"yarn:yarn.nodemanager.resource.memory-mb" : 15360,"yarn:yarn.scheduler.maximum-allocation-mb" : 15360},
        subnetwork_uri="projects/project-id/regions/us-central1/subnetworks/dataproc-subnet",
        retries= 1,
        retry_delay=timedelta(minutes=1),

    ) #starts a dataproc cluster


stop_cluster_example = DataprocClusterDeleteOperator(
    dag=dag,
    task_id='stop_cluster_example',
    cluster_name='example-{{ ds }}',
    project_id="your-project-id",
    region="us-central1",
    ) #stops a running dataproc cluster
    
{% endraw %}
```

Dataproc cluster create operator is yet another way of creating cluster and makes the same ReST call behind the scenes as a gcloud dataproc cluster create command or GCP Console. Stop cluster takes existing cluster's name and deletes the cluster. Airflow uses Jinja templating and parses `{{ ds }}` as execution date in YYYYMMDD format wherever used, so we can create cluster names based on when it was created and what data it is processing to have a better management and insight. 

local ssds -