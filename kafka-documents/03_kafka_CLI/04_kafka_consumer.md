# Kafka Console Consumer Guide

## **Introduction**

In this guide, you'll learn how to use the **Kafka Console Consumer CLI** to:

- Consume messages from Kafka topics
- Read messages from the beginning or only new messages
- View keys, values, timestamps, and partition info

---

## **Consuming from a Topic**

To start consuming messages from a topic (e.g., `second-topic`):

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second-topic
```

If the topic has **no messages**, the command will appear idle. To test, start a **console producer** in another terminal and send messages.

---

## **Creating a Multi-Partition Topic**

Create a topic with multiple partitions to test distributed message consumption:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic second-topic --partitions 3 --replication-factor 1
```

---

## **Using Round Robin Partitioner in Producer (this did not work, all the messages were going to same partition)**

To evenly distribute messages across partitions:

```sh
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic second_topic \
  --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner

```

> ⚠️ This partitioner is **inefficient for production** but useful for learning.

Now type messages like:

```
Hello World
My name is Stephane
It's working
```

They will appear in the consumer.

---

## **Consuming from the Beginning**

To see all previously sent messages:

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second-topic --from-beginning
```

### Note on Ordering

Messages may **not appear in the order** they were sent due to distribution across partitions.

- Kafka ensures **ordering within a partition**, not across partitions.

To confirm:

```sh
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic first-topic \
  --from-beginning
```

For a single-partition topic (`first-topic`), the order will be preserved.

---

## **Viewing Partition Info and Metadata**

Use the message formatter to display timestamp, key, value, and partition:

```sh
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic second-topic \
  --from-beginning \
  --property print.timestamp=true \
  --property print.key=true \
  --property print.value=true \
  --property print.partition=true
```

### Example Output:

```
CreateTime:1742630991003	Partition:0	null	g
CreateTime:1742631356308	Partition:0	null	a
CreateTime:1742631356958	Partition:0	null	b
CreateTime:1742631357582	Partition:0	null	c
CreateTime:1742631358062	Partition:0	null	d

```

This confirms that ordering is **per partition**.

---

## **Observing in UI (Optional)**

In tools like Conduktor:

- Navigate to the topic (e.g., `second-topic`)
- Filter messages by partition to observe distribution

---

## **Further Testing**

Continue sending messages:

```sh
another one
yet another
last one
```

You'll see them consumed and distributed across partitions (e.g., partition 0, 1, 2).

---

## **Conclusion**

You’ve learned to:

- Start the Kafka console consumer
- Read from the tail or the beginning of a topic
- Understand message distribution across partitions
- Use message formatting to view detailed metadata

This sets the stage for more advanced concepts like **consumer groups and parallel consumption**, which you'll explore next.
