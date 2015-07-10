# hdinsight-spark-examples
Examples to demonstrate Apache Spark on Azure HDInsight

## Spark Streaming
All Spark Streaming examples are in sparkstreaming folder.

### Prerequisites
In order to build and run the examples, you need to have:

1. Java 1.7/1.8 SDK.
2. Maven 3.x
3. Scala 2.10

### Build source code
You need the spark-streaming-eventhubs jar to build the example. While we are working to push the source code of spark-streaming-eventhubs to Apache Spark trunk, we provide a jar in the lib folder and a cmd command to install the jar to your local maven cache.

Follow these steps to build the examples:
1. push_mvn_cache.cmd (You only need to do this once)
2. build.cmd

### Run on HDInsight Spark cluster
For prototype, you can use notebook experience such as Zeppelin to run your code on HDInsight Spark cluster.

If you want to deploy production streaming pipeline, with HA and resiliency support, you need to:

1) Copy the jar-with-dependencies.jar in the target folder to the Storage Container associated with the HDInsight Spark cluster. 

You can do it using tools like Cloud Explorer. Or you can RDP to the cluster, copy the jar over to the headnode of the cluster, then run the following command:

  hadoop fs -copyFromLocal eventhubs-examples-eventcount-0.1.0-jar-with-dependencies.jar /sparktest

2) RDP to the cluster and run the following command:

  %SPARK_HOME%\bin\spark-submit.cmd --deploy-mode cluster --supervise --class org.apache.spark.streaming.eventhubs.example.EventCount wasb:///sparktest/eventhubs-examples-eventcount-0.1.0-jar-with-dependencies.jar checkpointDirectory policyName policyKey namespace name partitionCount consumerGroup outputDirectory

#### Cluster Mode
This command submit the Spark Streaming application in cluster mode with supervise. This means the driver will be run on the worker node and the sparkmaster will restart the driver in the case of driver crash.

Also, the example code enabled checkpointing, a feature provided by Spark Streaming, to checkpoint StreamingContext and other data so that driver can be restarted on a different node.

#### Using ReliableEventHubsReceiver
You can choose to use ReliableEventHubsReceiver with the same source code. You just need to turn on WriteAheadLog in Spark Configuration. One way to do this is to add the configuration in the cmd command:

  %SPARK_HOME%\bin\spark-submit.cmd --conf "spark.streaming.receiver.writeAheadLog.enable=true" --class ...

### Resource Constraints
A Spark Streaming application uses the same number of EventHubsReceiver instances as the number of EventHubs partitions. Each EventHubsReceiver instance needs 1 core to execute. And the application needs some cores to process the data. So we recommend at least 2 * (partitionCount) cores allocated to your Spark Streaming application. If you don't have enough cores left, your application may hang and wait for more resources.

What this means for HDInsight Spark cluster is that for a default configuration, a 4 node cluster can handle at most 8 EventHubs partitions (each node has 4 cores, and 16 cores can only handle at most 8 partitions).