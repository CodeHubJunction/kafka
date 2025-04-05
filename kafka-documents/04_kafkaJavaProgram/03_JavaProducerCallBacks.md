# Kafka Producer Callbacks & Sticky Partitioner (Java API)

**Speaker:** Stephane from Conduktor

---

## 🚀 Overview

In this lecture, we will:

- Explore how **callbacks** work in Kafka's Java Producer API
- Understand **record metadata** (topic, partition, offset, timestamp)
- Observe Kafka's **StickyPartitioner** and its behavior
- Learn how batch size and partitioner configuration affects data distribution

---

## ✅ Setting Up Callback in Producer

1. Duplicate your existing `ProducerDemo` class as `ProducerDemoWithCallback`.
2. Add a **callback** to the `producer.send()` method:

```java
producer.send(producerRecord, new Callback() {
    public void onCompletion(RecordMetadata metadata, Exception e) {
        if (e == null) {
            log.info("Received new metadata:\n" +
                "Topic: " + metadata.topic() + "\n" +
                "Partition: " + metadata.partition() + "\n" +
                "Offset: " + metadata.offset() + "\n" +
                "Timestamp: " + metadata.timestamp());
        } else {
            log.error("Error while producing", e);
        }
    }
});
```

- This logs partition, offset, and timestamp for each record.
- If an error occurs, it's logged via `log.error`.

---

## 🔄 Sending Multiple Messages

```java
for (int j = 0; j < 10; j++) {
				ProducerRecord<String, String> producerRecord = new ProducerRecord<>("demo_java", "hello world");

				// send data
				producer.send(producerRecord, new Callback() {

					@Override
					public void onCompletion(RecordMetadata metadata, Exception exception) {
						// TODO Auto-generated method stub

						if (exception == null) {
							log.info("Received new metadata \n" + "Topic: " + metadata.topic() + "\n" + "Partition: "
									+ metadata.partition() + "\n" + "Offset: " + metadata.offset() + "\n"
									+ "Timestamp: " + metadata.timestamp());
						} else {
							log.error("Error while producing", exception);
						}

					}
				});
		}
```

- This sends 10 messages with incrementing content.
- All use the same callback.
- Output shows metadata info per message.

# Kafka Producer Logs Explanation

## 🔍 Log Output Sample

```
kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 4
Offset: 77
Timestamp: 1743335144544
```

## 💬 Log Breakdown

| Part                           | Meaning                                         |
| ------------------------------ | ----------------------------------------------- | --------------------------------------------------- |
| `kafka-producer-network-thread | producer-1`                                     | Kafka's internal thread handling network operations |
| `ProducerDemoWithCallBack`     | Your Java class name                            |
| `Topic: demo_java`             | Kafka topic the message was sent to             |
| `Partition: 4`                 | Partition the message was assigned to           |
| `Offset: 77`                   | Message's position in the partition (see below) |

## 🧠 Understanding Offset

Yes, you are correct!

- **Offset** is the position of the message **within a partition**, not across the whole topic.
- So, `Offset: 77` means **there are 77 messages before it in Partition 4** (offsets start from 0).
- Each partition maintains its own offset sequence independently.

## 📦 Why All Messages Went to Partition 4?

You created messages like this:

```java
new ProducerRecord<>("demo_java", "hello world")
```

- **No key was provided**, so Kafka used the **default partitioner**.
- Since Kafka 2.4, the default partitioner uses **sticky partitioning** for messages without keys.

### 🧩 Sticky Partitioner Behavior

- Kafka uses **StickyPartitioner** by default (when no key is set).
- This improves performance by batching messages into the **same partition**.
- You may notice all 10 messages go to the **same partition** (e.g., partition 1).
- Picks a partition randomly
- Sends all messages to that partition **until the batch is full** or `linger.ms` time is up
- Then may switch to another partition

So, it's expected that your 10 quick messages went to **Partition 4**.

## ⏱️ Simulate Partition Switching with Batching

To observe sticky partitioning more clearly:

1. Send multiple batches.
2. Use delays between batches.
3. Reduce batch size.

## ⚙️ Configuration Reference

```java
properties.setProperty("batch.size", "400");
```

### Example:

```java
for (int j = 0; j < 10; j++) {

			for (int i = 0; i < 30; i++) {
				ProducerRecord<String, String> producerRecord = new ProducerRecord<>("demo_java", "hello world");

				// send data
				producer.send(producerRecord, new Callback() {

					@Override
					public void onCompletion(RecordMetadata metadata, Exception exception) {
						// TODO Auto-generated method stub

						if (exception == null) {
							log.info("Received new metadata \n" + "Topic: " + metadata.topic() + "\n" + "Partition: "
									+ metadata.partition() + "\n" + "Offset: " + metadata.offset() + "\n"
									+ "Timestamp: " + metadata.timestamp());
						} else {
							log.error("Error while producing", exception);
						}

					}
				});

			}

			try {
				Thread.currentThread().sleep(500);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

		}
```

- Sends 10 batches of 30 messages each.
- Waits 500ms between batches.
- Helps observe partition switching in logs.

- This means Kafka will try to **batch messages up to 400 bytes** per partition before sending them.
- If 400 bytes is not reached, Kafka may still send messages based on other settings (e.g., `linger.ms`).

# Kafka Producer Logs Explanation with `batch.size=400`

This document explains the Kafka producer behavior and log output when `batch.size` is set to `400`. It also helps interpret why messages may go to the same or different partitions even when `RoundRobinPartitioner` is used.

---

## ⚙️ Configuration Reference

```java
properties.setProperty("batch.size", "400");
```

- This means Kafka will try to **batch messages up to 400 bytes** per partition before sending them.
- If 400 bytes is not reached, Kafka may still send messages based on other settings (e.g., `linger.ms`).

---

## 📜 Sample Log Output

```
[kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 1
Offset: 836
Timestamp: 1743336007057

[kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 1
Offset: 837
Timestamp: 1743336007058

[kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 3
Offset: 188
Timestamp: 1743336007058

[kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 3
Offset: 189
Timestamp: 1743336007059

[kafka-producer-network-thread | producer-1] INFO ProducerDemoWithCallBack - Received new metadata
Topic: demo_java
Partition: 3
Offset: 190
```

---

## 🧠 What's Happening?

- Some messages go to **Partition 1**
- Others go to **Partition 3**
- But **multiple messages go to the same partition consecutively**

### ✅ This is **expected** due to batching behavior.

Kafka batches messages **per topic-partition**. When:

- `batch.size = 400`
- You send multiple small messages quickly
- Kafka may fill up a batch for one partition first (e.g., Partition 1)
- Then start a new batch for another partition (e.g., Partition 3)

So multiple messages can go to the **same partition** if they are grouped into the same batch.

---

## 🔍 Partition vs Offset Recap

- **Partition:** Identifies which Kafka partition the message went to.
- **Offset:** Sequential number of a message **within a partition**.

Example:

```
Partition: 3
Offset: 190
```

➡️ This message is the **191st** in **Partition 3** (since offset starts at 0).

---

## RoundRobin

batch size is not set..

```
properties.setProperty("partitioner.class", RoundRobinPartitioner.class.getName());
```

We can see the messages getting distributed to different partitions.

## So why are multiple messages still going to the same partition? Because of message batching:

    Kafka groups messages that are headed to the same topic and partition into a batch to optimize performance.

    When using round-robin, Kafka assigns a partition per batch, not necessarily per message.

    So if you're sending 30 messages in a tight loop and batching is active (due to batch.size, linger.ms, etc.), you might get multiple messages in the same batch, all going to the same partition.

    Once the batch is full or linger.ms is exceeded, the next batch will go to the next partition, and so on.
