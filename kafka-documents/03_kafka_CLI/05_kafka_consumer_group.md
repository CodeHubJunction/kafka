# Kafka Consumer Groups Explained

## **Introduction**

This guide demonstrates how to use the **Kafka Console Consumer CLI** within **consumer groups** to:

- Distribute message processing across multiple consumers
- Understand how partitions are assigned to consumers
- Handle rebalancing scenarios
- Work with committed offsets

---

## **Step 1: Create a Multi-Partition Topic**

To begin, create a fresh topic with three partitions:

```sh
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic third-topic --partitions 3 --replication-factor 1
```

---

## **Step 2: Start a Consumer in a Group**

Start a consumer and assign it to a consumer group:

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --group my-first-application
```

Nothing will be displayed yet because no messages have been produced.

---

## **Step 3: Start a Producer**

Open a new terminal and start a console producer using the round-robin partitioner:

```sh
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic third-topic \
  --property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner
```

Type some messages:

```
test
```

Messages will appear in the consumer.

---

## **Step 4: Add More Consumers to the Group**

Start a second and third consumer with the same group:

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --group my-first-application
```

Now, each consumer will be assigned partitions. If you send:

```
hello
world
last
```

Messages will be distributed among consumers. For example:

- Consumer 1 receives: `hello`, `last`
- Consumer 2 receives: `world`

Add a third consumer to see even distribution with three consumers and three partitions:

```
one
two
three
```

Each consumer receives one message.

---

## **Step 5: Add a Fourth Consumer (Overload)**

If you add a fourth consumer to the same group, one consumer won't receive any data because there are only three partitions.

You can confirm this by producing more messages:

```
a
b
c
```

Then stop one consumer and observe a **rebalance**. Continue producing:

```
d
e
f
```

Messages will be reassigned across remaining consumers.

---

## **Step 6: Consumer Offset Behavior**

If no consumers are running and you keep producing:

```
j
k
```

These messages are stored. When you restart a consumer from the same group (`my-first-application`), it will consume the missed messages (`j`, `k`).

---

## **Step 7: Using `--from-beginning` with a New Group**

To consume **all messages from the start**, use a new group and the `--from-beginning` flag:

```sh
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic third-topic \
  --group my-second-application \
  --from-beginning
```

All messages from the topic will be consumed.

### ⚠️ Important Note

Running the same command again **won’t re-read** from the beginning because the group has already committed offsets. `--from-beginning` only works **when the group reads the topic for the first time**.

---

## **Conclusion**

You’ve learned:

- How Kafka consumers operate in **groups**
- How Kafka **assigns partitions** to consumers
- How **rebalancing** works when consumers join/leave
- How to read **missed messages** and use `--from-beginning`

This is a core concept in Kafka and essential for building scalable, distributed applications. 🚀
