# Kafka Topics CLI

## Introduction

In this section, we will learn how to manage Kafka topics using the **Kafka Topics CLI**. This includes:

- Creating Kafka topics
- Listing Kafka topics
- Describing Kafka topics
- Increasing the number of partitions
- Deleting Kafka topics (Note: Avoid deleting topics on
  **Windows non-WSL2**, as it may cause crashes.)

## Running Kafka Topics Commands

You can check if the Kafka CLI is working by running:

```sh
kafka-topics.sh  # Linux/Mac/WSL2
kafka-topics.bat  # Windows (non-WSL2)
kafka-topics      # Homebrew/apt installations
```

If the command returns a **help blurb**, it means your installation is correct.

## Creating a Topic

To create a Kafka topic, run:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic first-topic
```

To specify the number of partitions:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic second-topic --partitions 5
```

For **secure clusters**, include the `--command-config` option:

```sh
kafka-topics.sh --command-config playground.config --bootstrap-server <server> --create --topic secure-topic --partitions 5
```

## Replication Factor

Replication Factor determines how many brokers will store a copy of your topic. Ensure that it does not exceed the number of available brokers.

Example:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic replicated-topic --replication-factor 1
```

### Understanding Kafka Topic Description Output

#### **📝 Breakdown of the Output**

When running the following command:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic replicated-topic1
```

Kafka provides detailed information about the specified topic. Below is an example output and its explanation:

```
Topic: replicated-topic1    TopicId: jPlOuUcTTraKRwvFpud6Mg    PartitionCount: 1    ReplicationFactor: 2    Configs:
```

### **Field Explanations**

- **Topic:** `replicated-topic1` → The name of the Kafka topic.
- **TopicId:** `jPlOuUcTTraKRwvFpud6Mg` → A unique identifier assigned to the topic by Kafka.
- **PartitionCount:** `1` → The topic has **only one partition**.
- **ReplicationFactor:** `2` → The partition is **replicated across two brokers**.
- **Configs:** _(empty)_ → No additional configurations have been set for this topic.

---

## **🗂 Partition Details**

```
Topic: replicated-topic1    Partition: 0    Leader: 2    Replicas: 2,0    Isr: 2,0    Elr: N/A    LastKnownElr: N/A
```

### **Field Explanations**

- **Partition:** `0` → This is **partition 0** (since there is only one partition for this topic).
- **Leader:** `2` → **Broker 2 is the leader** for this partition. The leader handles **all read and write operations**.
- **Replicas:** `2,0` → This partition is **replicated on Broker 2 and Broker 0**.
- **ISR (In-Sync Replicas):** `2,0` → **Both Broker 2 and Broker 0 are in sync** with the leader.
- **ELR (End Log Replica):** `N/A` → Kafka **does not track ELR** in this output.
- **LastKnownElr:** `N/A` → Similar to ELR, this field is **not applicable in this case**.

---

## **🔍 What This Means**

- ✅ **Your topic exists and is properly set up**.
- ✅ **The replication factor is `2`**, meaning data is stored on **two brokers (2 and 0)**.
- ✅ **Broker `2` is the leader**—all read/write operations go through it.
- ✅ **Both brokers (`2` and `0`) are in sync**—data is successfully replicated.
- ✅ **No errors are indicated in the output**, meaning replication is working fine.

This output confirms that your Kafka topic is functioning correctly with proper **leader election, replication, and partition management**.

## Listing Topics

To list all Kafka topics:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

## Describing a Topic

To get details of a specific topic:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic first-topic
```

This displays information about partitions, leaders, replicas, and ISR (In-Sync Replicas).

## Deleting a Topic

To delete a topic:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic second-topic
```

**Warning:** Avoid deleting topics on **Windows non-WSL2**, as it may cause crashes.

## Conclusion

We have covered how to use the Kafka Topics CLI for topic management, both on **local hosts** and the **Conduktor platform**. Mastering these commands will help you manage Kafka effectively, especially when working without a UI.

# Note:

## Kafka Replication Factor and Broker Limitations

## **Understanding Replication Factor**

In Kafka, the **replication factor** determines how many copies of a topic's partitions are stored across brokers for **fault tolerance**. The **number of brokers must be greater than or equal to the replication factor**.

For example:

- If there are **3 brokers**, you can set a replication factor up to **3**.
- If there is **only 1 broker**, the replication factor must be **1**.

## **What Happens If the Replication Factor Is Too High?**

If you attempt to create a topic with a **replication factor higher than the number of brokers**, you will receive the following error:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic replicated-topic1 --replication-factor 2
```

Error output:

```sh
Error while executing topic command : Replication factor: 2 larger than available brokers: 1.
[2025-03-03 10:57:19,659] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 2 larger than available brokers: 1.
(org.apache.kafka.tools.TopicCommand)
```

### **Why Does This Error Occur?**

- The command specifies a **replication factor of 2**.
- However, the Kafka cluster has **only 1 broker** running.
- Kafka requires **at least 2 brokers** to replicate the topic across them.
- Since there aren’t enough brokers, Kafka throws an `InvalidReplicationFactorException`.

## **How to Fix This Error**

### **Option 1: Reduce the Replication Factor (Single-Broker Setup)**

If you only have **one broker**, set the replication factor to `1`:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic replicated-topic1 --replication-factor 1 --partitions 3
```

### **Option 2: Add More Brokers (Multi-Broker Setup)**

If you want to use a replication factor **greater than 1**, start additional brokers. Example:

1. Start additional Kafka brokers (`broker-2`, `broker-3`).
2. Check the number of brokers in the cluster:

   ```sh
   kafka-broker-api-versions.sh --bootstrap-server localhost:9092
   ```

3. Retry the topic creation command with a valid replication factor.

## **Conclusion**

- Kafka requires **at least as many brokers as the replication factor**.
- If there are **not enough brokers**, Kafka will **reject** the topic creation.
- To resolve this issue, **either reduce the replication factor** or **add more brokers**.
