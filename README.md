# Create instance, table and column family
This project use Google Cloud as a deployed infrastructure. Use the built-in Cloud Shell, which you can open by clicking the "Activate Cloud Shell" button to start creating the project.

Set the following environment variables to make copying and pasting the codelab commands

```
INSTANCE_ID="bus-instance"
CLUSTER_ID="bus-cluster"
TABLE_ID="bus-data"
CLUSTER_NUM_NODES=3
CLUSTER_ZONE="us-central1-c"
```

Enable the Cloud Bigtable APIs by running this command.

```
gcloud services enable bigtable.googleapis.com bigtableadmin.googleapis.com
gcloud services enable dataflow.googleapis.com
```

Create an instance by running the following command (please be carefull, it will start charging immediately)

```
gcloud bigtable instances create $INSTANCE_ID \
    --cluster=$CLUSTER_ID \
    --cluster-zone=$CLUSTER_ZONE \
    --cluster-num-nodes=$CLUSTER_NUM_NODES \
    --display-name=$INSTANCE_ID
```

After successfully created the instance, populate the cbt configuration file and then create a table and column family by running the following commands:

```
echo project = $GOOGLE_CLOUD_PROJECT > ~/.cbtrc
echo instance = $INSTANCE_ID >> ~/.cbtrc

cbt createtable $TABLE_ID
cbt createfamily $TABLE_ID cf
```

# Import dataset
Run the following commands to import the table.

```
NUM_WORKERS=$(expr 3 \* $CLUSTER_NUM_NODES)
gcloud beta dataflow jobs run import-bus-data-$(date +%s) \
--gcs-location gs://dataflow-templates/latest/GCS_SequenceFile_to_Cloud_Bigtable \
--num-workers=$NUM_WORKERS --max-workers=$NUM_WORKERS \
--parameters bigtableProject=$GOOGLE_CLOUD_PROJECT,bigtableInstanceId=$INSTANCE_ID,bigtableTableId=$TABLE_ID,sourcePattern=gs://cloud-bigtable-public-datasets/bus-data/*
```

# Running code
Changing env to Java 11

```
sudo update-java-alternatives -s java-1.11.0-openjdk-amd64 && export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
```

Download the source code to running instance

```
git clone https://github.com/hacba98/de-final-project.git
cd de-final-project
```

Running simple lookup
```
mvn package exec:java -Dbigtable.projectID=$GOOGLE_CLOUD_PROJECT \
-Dbigtable.instanceID=$INSTANCE_ID -Dbigtable.table=$TABLE_ID \
-Dquery=lookupVehicleInGivenHour
```

Perform a route scanning
```
mvn package exec:java -Dbigtable.projectID=$GOOGLE_CLOUD_PROJECT \
-Dbigtable.instanceID=$INSTANCE_ID -Dbigtable.table=$TABLE_ID \
-Dquery=scanEntireBusLine
```

Perform a full bus routes and vehicles scan with an hour
```
mvn package exec:java -Dbigtable.projectID=$GOOGLE_CLOUD_PROJECT \
-Dbigtable.instanceID=$INSTANCE_ID -Dbigtable.table=$TABLE_ID \
-Dquery=scanManhattanBusesInGivenHour
```

# FAQ
[Q] Cannot compile the source code?

[A] The source code need Apache Maven to compile, normal Google cloud instance should have this tools. If it does not have, please install the latest Maven version.