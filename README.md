# Apache Storm on HDInsight 3.6 writing to Cosmos DB (Mongo DB API)

This is a basic example of using the storm-mongodb bolt (part of the base Apache Storm 1.1.0 distribution) to write to Cosmos DB (Mongo DB API) from Storm on HDInsight.

## How it works

To make this work, you'll need:

* An Azure subscription.

* A Storm on HDInsight 3.6 cluster.

* A Cosmos DB, using the Mongo DB API.

* The connection string for the Cosmos DB, Mongo DB.

* A pre-created database and collection in Cosmos DB.

The `resources/writer.yaml` topology emits random weather data (temperature, humidity, co2 level). This is passed to the storm-mongodb bolt, which writes it to the Mongo DB API exposed by Cosmos DB on Azure.

## Confgure and build

1. Fork & clone the repository so you have a local copy.

2. In the `dev.properties` file, change the `mongodb.url` to the connection string for Cosmos DB. Also change the `mongodb.collection.name` to the name of the collection.

    IMPORTANT!

    You have to modify the connection string to specify the database. This is apparently the only way you can specify the database to the storm-mongodb component. Add the database name between `/?` at the end of the connection string. For example, `...documents.azure.com:10255/mydatabase?ssl-true...`


3. Use `mvn package` to build everything.

    Once the build completes, the `target` directory will contain a file named `CosmosDBMongoDBAPI-1.0-SNAPSHOT.jar`.

## Test locally

Since this just writes to Cosmos DB, you can test it locally if you have a [Storm development environment](http://storm.apache.org/releases/current/Setting-up-development-environment.html). Use the following steps to run locally in the dev environment:

```bash
storm jar CosmosDBMongoDBAPI-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /writer.yaml --filter dev.properties
```

Output is logged to the console when running locally. Use __Ctrl+C__ to stop the topology.

## Deploy

1. Use SCP to copy the jar package to your HDInsight cluster. Replace USERNAME with the SSH user for your cluster. Replace CLUSTERNAME with the name of your HDInsight cluster:

    ```bash
    scp ./target/CosmosDBMongoDBAPI-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:EventHubExample-1.0-SNAPSHOT.jar
    ```

    For more information on using `scp` and `ssh`, see [Use SSH with HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix).

2. Use SCP to copy the dev.properties file to the server:

    ```bash
    scp dev.properties USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:dev.properties
    ```

2. Once the file has finished uploading, use SSH to connect to the HDInsight cluster. Replace **USERNAME** the the name of your SSH login. Replace **CLUSTERNAME** with your HDInsight cluster name:

    ```bash
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Use the following commands to start the topologies:

    ```bash
    storm jar CosmosDBMongoDBAPI-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writer.yaml --filter dev.properties
    ```

    This will start the topology using a friendly name of "mongodbwriter".

4. To view the logged data, go to https://CLUSTERNAME.azurehdinsight.net/stormui, where __CLUSTERNAME__ is the name of your HDInsight cluster. Select the topologies and drill down to the components. Select the __port__ entry for an instance of a component to view logged information.

5. Use the following commands to stop the reader:

    ```bash
    storm kill mongodbwriter
    ```

## Project code of conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.