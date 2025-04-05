# Kafka Producer Tutorial (Java API)

**Speaker:** Stephane from Conduktor

---

## 🌟 Objective

In this lecture, you'll learn how to:

- Write your first **Kafka producer** using the **Java API**
- Send data to Apache Kafka
- View basic configuration parameters
- Confirm that data is received in a **Kafka Console Consumer**

---

## 🛠️ Steps to Create a Kafka Producer

1. **Create Producer Properties**
2. **Create the Producer**
3. **Send Data**
4. **Flush and Close the Producer**

---

## Before Running the Code

### Create the Topic

- Create `demo_java` topic:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic demo_java
```

- Check if the topic is created:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

- Start consumer:

```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo_java

```

---

## 🖥️ Verifying the Result

### Option 1: View in Conduktor Console

- Navigate to topic `demo_java`
- You should see:
  ```
  null    hello world
  ```

### Option 2: Use kafka-console-consumer

```bash
kafka-console-consumer.sh --bootstrap-server <your-server> \
--topic demo_java --from-beginning
```

## 🔧 Setting up the Logger

```java
private static final Logger log = LoggerFactory.getLogger(ProducerDemo.class.getSimpleName());
```

- Use `LoggerFactory` from `org.slf4j`.

### Replace System Output

```java
log.info("Hello World");
```

- Run the program. You should see `producer demo, hello world` if everything is working.
- If not, check `build.gradle` and:
  - Replace `testImplementation` with `implementation`
  - Ensure dependencies: `slf4j-api` and `slf4j-simple` are added.

---

## 🗞️ Create Producer Properties

```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "127.0.0.1:9092");
```

> Connects to a Kafka broker running locally.

---

## 🌐 Using Conduktor Playground (Remote Kafka Cluster)

- Copy connection config from your Playground (UI).
- Include:

```java
properties.setProperty("security.protocol", "SASL_SSL");
properties.setProperty("sasl.jaas.config", "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"<USERNAME>\" password=\"<PASSWORD>\";");
properties.setProperty("sasl.mechanism", "PLAIN");
properties.setProperty("bootstrap.servers", "<your-playground-bootstrap-server>");
```

> Escape characters will auto-format inside IntelliJ when pasting.

- You can comment/uncomment the local/remote connection configs as needed.

---

## ⚙️ Serialization Setup

Kafka requires serialization for both key and value:

```java
properties.setProperty("key.serializer", StringSerializer.class.getName());
properties.setProperty("value.serializer", StringSerializer.class.getName());
```

> Ensures strings are serialized into bytes before sending to Kafka.

---

## 🧱 Create Kafka Producer

```java
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
```

- Key and Value types are both `String` to match the serializers.

---

## 📦 Create Producer Record

```java
ProducerRecord<String, String> producerRecord = new ProducerRecord<>("demo_java", "hello world");
```

- `"demo_java"` is the topic name.
- `"hello world"` is the value sent.

---

## 🚀 Send Data to Kafka

```java
producer.send(producerRecord);
```

- **Asynchronous** call.

```java
producer.flush(); // Block until all data is sent
producer.close(); // Also calls flush()
```

> Without flush/close, the program may exit before data is sent.

---

output in kafka consumer

- Output:
  ```
  hello world
  ```

---

## ✅ Summary

- You successfully:
  - Wrote a Kafka producer in Java
  - Configured it for both local and remote clusters
  - Serialized and sent a simple message
  - Verified delivery using consumer tools

---

## 📚 Next

See you in the next lecture for more advanced Kafka usage!
