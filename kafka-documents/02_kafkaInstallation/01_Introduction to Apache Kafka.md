# Setting up Kafka

## Steps

1. Download the latest Kafka from the [link](<[https://kafka.apache.org/downloads)>) download Linux x64 Compressed Archive.

2. Go to the directory .

   ```sh
    cd /home/raptor/Software
   ```

3. Extract the Kafka binary here

   ```sh
    sudo tar -xvzf kafka_2.12-3.9.0.tgz
   ```

4. Change the permission of the directrories and its sub-directories

   ```sh
    sudo chmod -R 777 kafka_2.12-3.9.0
   ```

5. Setting Up JDK by sourcing the `.bashrc`:

   ```sh
    source ~/.bashrc
   ```

6. Select the jdk

   ```sh
   jdk17
   ```

7. Setting Up Kafka Environment Variables

   edit `.bashrc`

   ```sh
   nano ~/.bashrc
   ```

   edit these to bashrc and add java_home and path

   ```sh
    alias jdk17="export JAVA_HOM

    alias kafka39="\
    export KAFKA_HOME='/home/raptor/Software/kafka_2.12-3.9.0'; \
    export PATH=\$KAFKA_HOME/bin:\$PATH; \
    echo \"Now using Kafka 3.9 at \$KAFKA_HOME\"; \
    echo 'To set up kafka'; \
    echo 'kafka39 -> kafka_2.12-3.9.0'; \
    "

    echo 'To set up kafka'
    echo 'kafka39 -> kafka_2.12-3.9.0'
   ```

8. To check if the configuration is correct sourcing `.bashrc` and then

   ```sh
   kafka-topics.sh
   ```

   if correctly configured, it will show the help.

9. To start zookeeper:
   ```sh
   zookeeper-server-start.sh ./kafka_2.12-3.9.0/config/zookeeper.properties
   ```
10. To start kafka server:

    ```sh
    kafka-server-start.sh ./kafka_2.12-3.9.0/config/server.properties
    ```

11. To stop, you need to first stop kafka server and then zookeeper

12. To edit kafka and zookeeper data storage directory. Edit, zookeeper.properties in config directory.

    ```sh
    # the directory where the snapshot is stored.
    dataDir=/tmp/zookeeper
    ```

    to change kafka log:

    ```sh
    # A comma separated list of directories under which to store log files
    log.dirs=/tmp/kafka-logs
    ```

# Setting up Kafka with out zookeeper.

## Steps

1. Go to the directory kafka and to generate uuid.

   ```sh
   kafka-storage.sh random-uuid
   Xi81Dp3KQPCx8AXyQauIZA
   ```

2. To format the metadata directory

   ```sh
   kafka-storage.sh format -t Xi81Dp3KQPCx8AXyQauIZA -c ./kafka_2.12-3.9.0/config/kraft/server.properties
   ```

3. To format the metadata directory

   ```sh
   kafka-storage.sh format -t Xi81Dp3KQPCx8AXyQauIZA -c ./kafka_2.12-3.9.0/config/kraft/server.properties
   ```

   output:

   ```sh
   Formatting metadata directory /tmp/kraft-combined-logs with metadata.version 3.9-IV0.
   ```

4. To start kafka server:

   ```sh
   kafka-server-start.sh ./kafka_2.12-3.9.0/config/kraft/server.properties
   ```
