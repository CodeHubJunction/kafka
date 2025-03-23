# Kafka Console Producer Guide

## **Introduction**

This guide explains how to use the **Kafka Console Producer** to send messages into Kafka topics. You will learn how to:

- Produce messages **with or without keys**
- Work with **secure and insecure clusters**
- Handle **topic auto-creation**
- Set **producer properties** for reliability

---

## **Basic Topic Creation**

Before producing data, create a topic (e.g., `first-topic`) with a single partition:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic first-topic --partitions 1 --replication-factor 1
```

This sets up a topic for your messages.

---

## **Launching Kafka Console Producer**

### **In a Secure Cluster**

```sh
kafka-console-producer.sh --producer.config playground.config --bootstrap-server <broker> --topic first-topic
```

### **In a Local Insecure Cluster**

```sh
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first-topic
```

When the Chevron prompt (`>`) appears, you're ready to type and send messages. Press `Enter` to send each one.

Eg:
Hello world
I love kafka

To exit, press `Ctrl + C`.

---

## **Producer Properties**

You can enhance producer behavior using the `--property` flag.

Example with acknowledgments:

```sh
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first-topic --property acks=all
```

This ensures that **all brokers** acknowledge each message.

---

## **Producing to Non-Existent Topics**

### **If Auto-Creation is Disabled (e.g., Conduktor)**

You will see an error like:

```
ERROR Error when sending message to topic new-topic: org.apache.kafka.common.errors.UnknownTopicOrPartitionException
```

### **If Auto-Creation is Enabled (e.g., Default Local Kafka)**

Messages will trigger the topic's creation, but you might see warnings:

```
WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 1 : {new-topic=LEADER_NOT_AVAILABLE}
```

After a few tries, the topic is created with default settings (e.g., 1 partition).

You can configure the default number of partitions in `config/server.properties`:

```properties
default.num.partitions=3
```

---

## **Producing with Keys**

To ensure that messages with the same key go to the same partition:

```sh
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first-topic \
  --property parse.key=true \
  --property key.separator=:
```

Example input:

```
name:Stephane
example-key:example-value
```

### Notes:

- The part **before** the colon (`:`) is the **key**.
- The part **after** is the **value**.

⚠️ If you omit the key separator, Kafka throws an exception.

---

## **Observing Messages in UI (Optional)**

If you're using a UI like Conduktor:

- View messages with or without keys
- Validate message delivery and partitioning

---

## **Conclusion**

You’ve now learned how to:

- Use the Kafka Console Producer
- Produce messages securely and reliably
- Work with keys and partitions
- Understand behavior with non-existent topics

This foundational knowledge will help you as you move deeper into Kafka. See you in the next lecture!
