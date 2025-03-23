# Kafka Consumer Groups CLI Guide

## **Introduction**

In this guide, you'll learn how to manage Kafka **consumer groups** using the Kafka CLI. Specifically, we'll cover how to:

- List existing consumer groups
- Describe a specific consumer group
- Understand consumer lag
- Observe partition assignment across consumers
- Handle temporary consumer groups

---

## **1. List Existing Consumer Groups**

Use the following command to list all active consumer groups:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

Example output:

```
my-first-application
my-second-application
```

These can also be viewed via the UI in tools like Conduktor.

---

## **2. Describe a Consumer Group**

To see details about a specific consumer group:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-second-application
```

Example output:

```
TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG  CONSUMER-ID   HOST         CLIENT-ID
third-topic   0          14              14              0    -             -            -
```

This means the group has **caught up** and has **no lag**.

---

## **3. Introduce Lag by Producing Messages**

Start a producer to send new messages:

```sh
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic third-topic
```

Send:

```
A
B
C
D
E
```

Then, re-run the describe command to observe **lag**:

```
CURRENT-OFFSET: 14 | LOG-END-OFFSET: 16 | LAG: 2
```

---

## **4. Consume and Reduce Lag**

Start a consumer in the same group to read the pending messages:

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --group my-second-application
```

Messages `A` to `E` will be consumed, and lag should drop to 0.

---

## **5. Observe Partition Assignment**

Start multiple consumers in the **same group**:

```sh
# Terminal 1
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --group my-second-application

# Terminal 2
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --group my-second-application
```

Now describe the group again:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-second-application
```

Each partition will be assigned to a different **consumer ID**:

- Consumer A: partitions 0 & 1
- Consumer B: partition 2

This demonstrates Kafka's ability to distribute partitions across consumers in the same group.

---

## **6. Using a Consumer Without a Group ID**

Run a consumer without specifying a group:

```sh
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic third-topic --from-beginning
```

These consumers will still appear briefly when listing groups:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

Temporary group IDs like `console-consumer-xxxx` may appear but will **disappear after a while**.

> ❗ Best Practice: Always specify a **group ID** for consistent tracking and offset management.

---

## **Conclusion**

You now know how to:

- List and describe consumer groups
- Monitor and interpret lag
- Understand partition assignment
- Handle temporary groups

Managing consumer groups is **essential for monitoring and scaling** Kafka consumers. See you in the next lecture! 🚀
