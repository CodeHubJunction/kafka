# Kafka Producer with Keys (Java API)

**Speaker:** Stephane from Conduktor

---

## 📈 Overview

In this lecture, we:

- Learn how to send **messages with keys** in Kafka
- Observe that **same keys are routed to the same partition**
- Use key-value pairs in `ProducerRecord`
- Run the code twice to verify consistent partitioning

---

## ✅ What Are Keys For?

- Kafka uses the key to **determine the target partition**.
- Same key = same partition (ensures message order per key).

Example:

- `truck_id_123` → always goes to **partition 0**
- `truck_id_345` → always goes to **partition 1**

---

# Kafka Producer Demo With Keys - Java Code Explained

This document explains a Java Kafka producer example that demonstrates how to send messages with keys, potentially to different Kafka partitions.

---

## 🧾 Purpose

The purpose of this code is to:

- Set up a Kafka producer
- Send messages to a Kafka topic (`demo_java`)
- Use keys to control message partitioning
- Log which key went to which partition

---

## 🛠 Imports

```java
import java.util.Properties;
import java.util.Random;
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

- These are the required classes for Kafka producer setup, logging, and creating producer records.

---

## 🧱 Class Setup

```java
public class ProducerDemoWithKeys {
    private static final Logger log = LoggerFactory.getLogger(ProducerDemoWithKeys.class);
```

- Sets up a logger to display output in the console.

---

## ⚙️ Configuration

```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "127.0.0.1:9092");
properties.setProperty("key.serializer", StringSerializer.class.getName());
properties.setProperty("value.serializer", StringSerializer.class.getName());
```

- Configures Kafka producer to connect to the Kafka broker running on `localhost:9092`.
- Specifies that both the key and value will be serialized as strings.

Commented-out lines show how to configure for remote Kafka clusters with authentication.

---

## 🚀 Producer Creation

```java
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
```

- Initializes the Kafka producer with the defined properties.

---

## 🔁 Message Loop

```java
Random random = new Random();
int[] numbers = { 1, 5, 233, 5466, 90890 };
```

- Defines a fixed set of numbers to randomly generate keys.

```java
for (int j = 0; j < 10; j++) {
    for (int i = 0; i < 5; i++) {
        int index = random.nextInt(numbers.length);
        String topic = "demo_java";
        String key = "id_" + numbers[index];
        String value = "hello world " + i;
```

- Nested loop sends multiple messages in batches with some delay.
- `key` is generated using a random number from the predefined list.
- The `value` is a simple message including the loop index.

---

## 📤 Sending the Record

```java
ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic, key, value);
```

- Creates a record with the specified topic, key, and value.

```java
producer.send(producerRecord, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception == null) {
            log.info("Key: " + key + " Partition: " + metadata.partition());
        } else {
            log.error("Error while producing", exception);
        }
    }
});
```

- Asynchronously sends the record.
- Logs the key and the partition it was sent to upon successful completion.

---

## ⏱ Delay Between Batches

```java
Thread.currentThread().sleep(1000);
```

- Introduces a 1-second delay between batches.

---

## 🧹 Cleanup

```java
producer.flush();
producer.close();
```

- `flush()` ensures all records are sent before shutting down.
- `close()` releases resources.

---

## 🧠 Learning Points

- Kafka uses the **key's hash** to determine the partition (if keys are specified).
- The same key always maps to the same partition.
- Randomizing the key helps observe partition distribution.
- Removing the key results in round-robin partitioning.

---

## ✅ Final Tip

Check how many partitions your topic has with:

```bash
kafka-topics.sh --create --topic demo_java --partitions 5 --replication-factor 1 --bootstrap-server 127.0.0.1:9092
```

- If there's only **1 partition**, all messages will obviously go to that one.

## ✅ Summary

- Kafka routes messages with the same key to the same partition
- Helps maintain **order for a key** across records
- You can verify partitioning behavior using logs or Conduktor UI
- Learned how to set keys in `ProducerRecord`

---

## 📚 Next

In the next lecture, we’ll explore how **custom partitioners** can alter this behavior.
