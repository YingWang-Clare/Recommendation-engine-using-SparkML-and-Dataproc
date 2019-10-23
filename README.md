# Recommendation-engine-using-SparkML-and-Dataproc
This is a learning project of Coursera Course: Google Cloud Platform Big Data and Machine Learning Fundamentals.

In this lab, we suppose that we have an existing on-premise architecture for housing rental recommendation. Our aim will be to migrate this infrastructure to GCP. Machine Learning is done using PySpark SparkML library.

## Notes
Cloud Dataproc is a fast, easy-to-use, fully managed cloud service for running Apache Spark and Apache Hadoop clusters in a simpler, more cost-efficient way.
## Create Cloud SQL instances
On GCP, create an MySQL instance. Set the proper name, and define a password.

Allocate 2 vCPUs to the Master and Workers.

Once the instance is created, access instance details. Then, connect to the instance using Cloud Shell.

Once a SQL shell connected to the instance is ready, we can type SQL queries.

## Create tables
* Three tables that we're gonna create:
    1. Accommodation table that describes the accommodation
    2. Rating table that describes the rating of each accommodation
    3. Recommendation table that attache recommendation to a user I

```SQL
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
    id varchar(255),
    title varchar(255),
    location varchar(255),
    price int,
    rooms int,
    rating float,
    type varchar(255),
    PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
    userId varchar(255),
    accoId varchar(255),
    rating int,
    PRIMARY KEY(accoId, userId),
    FOREIGN KEY (accoId) 
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
    userId varchar(255),
    accoId varchar(255),
    prediction float,
    PRIMARY KEY(userId, accoId),
    FOREIGN KEY (accoId) 
    REFERENCES Accommodation(id)
);

SHOW DATABASES;
```
Then, use `recommendation_spark` table.

## Store data in Google Cloud Storage
Next, we need to import data into those tables we just created.

Two options to do that:
1. Through GCP UI
2. Through GCP shell command line.

We'll use command line to achieve it. Open a new shell and type:

```
echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID

echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID

echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv
```

## Move data from Cloud Storage to Cloud SQL
Import `.csv` file on Storage into SQL instance. Select the correct database name and name them properly.

## Export data from Cloud SQL
We can go back to the SQL instance, connect to the instance via Cloud Shell. Then, we can access to the data by running SQL queries.

## Launch Dataproc
Computation and storage on GCP are split, so we need to allow Cloud Dataproc to connect with Cloud SQL.

Create a Cloud Dataproc cluster and name it properly.

Allow Dataproc to access this specific SQL cluster.

```
echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
CLUSTER=rentals
CLOUDSQL=rentals
ZONE=us-central1-a
NWORKERS=2

machines="$CLUSTER-m"
for w in `seq 0 $(($NWORKERS - 1))`; do
    machines="$machines $CLUSTER-w-$w"
done

echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
ips=""
for machine in $machines; do
    IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\[u'//g" | sed "s/'\]//g" )/32
    echo "IP address of $machine is $IP_ADDRESS"
    if [ -z  $ips ]; then
        ips=$IP_ADDRESS
    else
        ips="$ips,$IP_ADDRESS"
    fi
done

echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
gcloud sql instances patch $CLOUDSQL --authorized-networks $ips
```

## Load ML models:
Assume that the ML models are already built. 
```
gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
```

We also need to specify our Cloud SQL IP in this script.

Then we can copy the file to Cloud Storage Bucket.
```
gsutil cp train_and_apply.py gs://$DEVSHELL_PROJECT_ID
```

## Run Jobs on Dataproc
We can now run the ML job on Dataproc. Just choose submit.

We need to specify the name of the cluster and the job type as PySpark, and the URL of the main python file on the Cloud Storage.

## Check the result
After the job is done, we can check the result in the `Recommendation` table.


