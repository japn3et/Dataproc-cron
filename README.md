# Dataproc-git-cron
Hive Job scheduling from Onprem to Dataproc

This repository contains scripts to set up and run a data processing pipeline on Google Cloud Dataproc, aimed at migrating Hive and Spark jobs from an on-prem Hadoop environment to Google Cloud. The workflow involves creating a Dataproc cluster, managing data in HDFS, running Hive jobs, and scheduling daily jobs using Cron.

## Setup and Workflow

### Step 1: Create Dataproc Cluster

Run the following command to create a long-running Dataproc cluster with necessary configurations. Adjust the bucket and project names as needed:
```bash
gcloud dataproc clusters create <your-cluster-name> \
    --enable-component-gateway \
    --bucket <your-gcp-bucket> \
    --region us-central1 \
    --zone us-central1-a \
    --master-machine-type e2-standard-2 \
    --master-boot-disk-size 100 \
    --num-workers 2 \
    --worker-machine-type e2-standard-2 \
    --worker-boot-disk-size 100 \
    --image-version 2.1-rocky8 \
    --optional-components JUPYTER,ZEPPELIN \
    --max-idle 3600s \
    --project <your-project-id>
```

### Step 2: Configure HDFS Directory

SSH into the Dataproc master node to set up HDFS directories:
```bash
gcloud compute ssh <cluster-name>-m --zone us-central1-a --project <your-project-id>
```
Then, on the master node:
```bash
sudo su hdfs
hadoop fs -mkdir -p /user/<username>/project
hadoop fs -chmod -R 777 /user/<username>/
```

### Step 3: Prepare Data and Scripts in HDFS

Copy datasets and HQL scripts to the HDFS directory:
```bash
hadoop fs -cp -f gs://<common-bucket>/dataset/txns /user/<username>/project/
hadoop fs -cp -f gs://<common-bucket>/code/cust_etl.hql /user/<username>/project/
```

### Step 4: Run Hive Jobs with a Bash Script

A Bash script (`gcp_hive_schedule.sh`) is provided to automate data loading and transformation in Hive. The script:
1. Creates a managed Hive table.
2. Loads raw data from HDFS.
3. Runs an HQL script to insert data into a curated external table.

To execute the script manually:
```bash
bash /home/<username>/gcp_hive_schedule.sh
```

### Step 5: Schedule Job with Cron

Add the script to `crontab` to run daily:
```bash
crontab -e
```
Add the following line to schedule:
```bash
0 2 * * * /home/<username>/gcp_hive_schedule.sh
```
This setup automates the daily ETL tasks on Google Cloud Dataproc, simplifying the migration of on-prem Hadoop workflows to the cloud.
