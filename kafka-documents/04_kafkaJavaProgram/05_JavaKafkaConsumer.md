# Kafka Consumer Demo - Java Code Explanation and Learning Notes

This document summarizes a transcript of a Kafka consumer tutorial using Java. It walks through writing a basic consumer to receive data from a Kafka topic and explains key concepts and configurations.

---

## 🎯 Objective

- Write a basic Kafka **consumer** using Java
- Understand **polling**, **offsets**, **subscriptions**, and **consumer groups**
- Observe **data retrieval** and **efficiency of Kafka**

---

## 🧱 Consumer Setup

1. **Start with a new class**: `ConsumerDemo`
2. **Reuse Kafka connection properties** from the producer demo

```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "127.0.0.1:9092");
```

If connecting to a remote playground (e.g., Conduktor), you’ll add extra security configs.

---

## ⚙️ Consumer Configuration

- **Deserializers** (matching the producer’s serializers):

```java
properties.setProperty("key.deserializer", StringDeserializer.class.getName());
properties.setProperty("value.deserializer", StringDeserializer.class.getName());
```

- **Group ID**:

```java
String groupId = "my-java-application";
properties.setProperty("group.id", groupId);
```

- **Auto Offset Reset**:

```java
properties.setProperty("auto.offset.reset", "earliest");
```

- `none`: Fail if no offsets are committed. Please refer notes below.
- `earliest`: Read from the beginning of the topic.
- `latest`: Read only new messages from now.

---

## 🛠 Creating the Consumer

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
```

- Subscribing to a topic:

```java
String topic = "demo_java";
consumer.subscribe(Arrays.asList(topic));
```

---

## 🔁 Polling for Data

- Use a `while (true)` loop to continuously poll Kafka:

```java
while (true) {
    log.info("Polling");
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
```

- `poll(Duration)` fetches records. If records are available, they’re returned immediately. Otherwise, the method waits.

- Loop through received records:

```java
    for (ConsumerRecord<String, String> record : records) {
        log.info("Key: {} | Value: {}", record.key(), record.value());
        log.info("Partition: {} | Offset: {}", record.partition(), record.offset());
    }
}
```

---

## 📈 Consumer Behavior and Logging

- Initially, logs will show repeated `Polling` messages.
- After 3–4 polling cycles, messages will be received (if any are available) as consumer is as the consumer is getting configured.
  - like a consumer joining the group
  - identifying the partitions
  - finding the offset, etc.
- Kafka will assign available partitions to the consumer.
- If no committed offsets are found, `auto.offset.reset` determines where to start.

---

## 🔄 Batch Processing and Efficiency

- Kafka consumer can batch messages from a partition, pulling up to 1MB of data at once.
- Logs will show large bursts of messages from partitions like 0, 1, or 2.
- Efficient retrieval reduces load on Kafka brokers.

---

## 🔁 Re-running the Consumer

- If the consumer restarts with the same `group.id`, it **resumes from the last committed offsets**.
- If no new messages are sent, the consumer will poll and wait.
- Once new messages are produced, they’ll appear in logs.

---

## ❌ Shutdown Notes

- Current implementation does not handle **clean shutdown**.
- On abrupt exit, it takes time to **rejoin the group** and find committed offsets.
- Future enhancement: Add a mechanism to gracefully exit the loop.

---

## ✅ Summary

- We created a Kafka consumer using Java.
- Subscribed to a topic and read messages using `poll()`.
- Logged key, value, partition, and offset.
- Observed consumer efficiency and offset-based resume behavior.

---

### Notes:

# Kafka Consumer Configuration Discussion

## Overview

This document captures a discussion regarding the Kafka consumer configuration property
`auto.offset.reset` and the impact of using the value `none`.

---

## Conversation Transcript

### Question 1

> In a Kafka there is this configuration:
>
> ```properties
> properties.setProperty("auto.offset.reset", "none");
> ```
>
> What happens in this case?

**Answer**:  
If you set `auto.offset.reset = "none"`, then Kafka will **not** automatically reset the consumer’s offset position if there is no valid offset or if the requested offset is out of range. Instead, the consumer will throw an exception (for example, `OffsetOutOfRangeException`) and fail, rather than silently resetting to earliest or latest. This setting is often chosen for strict or debugging scenarios where you want to ensure no data is skipped or replayed unexpectedly.

---

### Question 2

> When the value is `none`, if we don't have any existing consumer group, it fails.

**Answer**:  
Yes, that’s correct. With `auto.offset.reset = "none"`, if the consumer group has no previously committed offsets (e.g., it’s a brand-new group), Kafka will find no existing offset for that group. Instead of automatically resetting, it fails immediately by throwing an exception. This behavior helps you detect and handle scenarios where no committed offset exists.

---

### Question 3

> How do we handle it in that case?

**Answer**:  
To handle the scenario where `auto.offset.reset = "none"` causes a failure for a brand-new consumer group (i.e., with no committed offsets), you have two main approaches:

1. **Use a different auto offset reset policy**  
   If you’re okay with letting the consumer begin reading from either the earliest or the latest offset, set `auto.offset.reset` to either `"earliest"` or `"latest"`. Kafka will automatically assign an initial offset instead of failing.

2. **Manually set or manage offsets**
   - Catch the exception (`OffsetOutOfRangeException`) in your application code and decide how to proceed (e.g., manually initialize your offsets).
   - Use an external offset store or custom logic to ensure that an offset is committed before the consumer starts so that the group has a valid offset to resume from.

In most practical use cases, `"earliest"` or `"latest"` is chosen to avoid failing consumers, but if you need strict or manual offset control, you can handle the exception as part of your logic.

---

### Question 4

> When do we use `none`?

**Answer**:  
Setting `auto.offset.reset = "none"` is relatively rare and is used when you want strict or deterministic offset management. Common reasons include:

1. **Regulatory/Compliance Requirements**  
   You may need to guarantee that no data is reprocessed or skipped without explicit acknowledgment. Failing fast ensures you never silently start reading from an unintended position.

2. **Debugging/Diagnostics**  
   If you’re investigating offset issues or want to ensure you never start at an unexpected position, you might prefer the consumer fails rather than silently resetting.

3. **Manual Offset Management**  
   In advanced scenarios, if you want to explicitly manage offsets outside Kafka or set them manually, failing immediately upon not finding an existing offset gives you full control over what happens next.

In general, most developers use `"earliest"` or `"latest"`, but `"none"` can be beneficial when strict offset control and failure on missing offsets are critical to your application’s guarantees.

---

### 📌 Next Step:

Implement clean shutdown logic to handle graceful consumer exit using a shutdown hook or interrupt handler.

---

Happy Kafka learning! 🚀
