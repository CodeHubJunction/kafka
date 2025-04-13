# Kafka Consumer Offsets and Auto-Commit Behavior

This document explains how **consumer offsets** are handled in Kafka, focusing on the **auto-commit mechanism**, **at-least-once delivery**, and how the Java Consumer API commits offsets behind the scenes.

---

## 🎯 Objective

- Understand how **offsets** are committed in Kafka consumers
- Learn the behavior of **auto-commit** and the **poll loop**
- Ensure **at-least-once processing** semantics in Java

---

## 🔁 What Are Offsets?

Kafka uses **offsets** to keep track of a consumer's progress in a partition. These offsets are:

- Stored **in Kafka** itself
- Used to **resume reading** from where a consumer left off

---

## ⚙️ Auto-Commit Basics

By default, Kafka consumers commit offsets **automatically**.

### Default Configuration:

```properties
enable.auto.commit=true
auto.commit.interval.ms=5000
```

- **`enable.auto.commit`**: Enables auto-commit behavior
- **`auto.commit.interval.ms`**: Offsets are committed every 5 seconds

### How It Works:

1. The consumer calls `poll()` to retrieve records.
2. Kafka starts an **internal commit timer**.
3. After `auto.commit.interval.ms` (e.g., 5 seconds), the **next `poll()` triggers an async commit**.

---

## ✅ Ensuring At-Least-Once Semantics

At-least-once means:

- Messages are **not lost**, but may be **processed more than once** in edge cases

To achieve this:

- Ensure **all records are processed** before calling `poll()` again

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));

    for (ConsumerRecord<String, String> record : records) {
        // ✅ Process the record here
    }

    // ✅ poll() is only called after processing
}
```

### ⚠️ If You Poll Before Processing:

- Offset might be committed **before** processing finishes
- Risk of **message loss** if consumer crashes

---

## 🔍 What Happens Behind the Scenes

1. You call `poll()` → Timer starts
2. Kafka returns some messages
3. Timer continues ticking...
4. On a later `poll()` (after interval passes), Kafka triggers an **async offset commit**
5. Timer resets

This cycle ensures offsets are regularly updated.

---

## 🧪 Example: Consumer With Auto-Commit

Using `ConsumerDemoWithShutdown`, assume:

- `enable.auto.commit=true`
- `auto.commit.interval.ms=5000`

### Flow:

1. `poll()` is called → Timer starts
2. Messages are received and processed
3. After 5 seconds, next `poll()` commits offsets **asynchronously**
4. Offsets reflect the **last batch** of successfully processed messages

✅ This is considered **at-least-once processing** as long as processing is done **before polling again**

---

## 📝 Summary

| Setting                    | Value       | Description                                   |
| -------------------------- | ----------- | --------------------------------------------- |
| `enable.auto.commit`       | `true`      | Enables automatic offset commits              |
| `auto.commit.interval.ms`  | `5000`      | Commits every 5 seconds after `poll()`        |
| Processing before `poll()` | ✅ Required | Ensures at-least-once semantics               |
| Commit Type                | Async       | Happens in the background via Kafka internals |

---

Kafka handles much of the offset management for you, but understanding when and how offsets are committed is key to building **reliable, fault-tolerant** consumers.

In the next lesson, you'll learn how to take **manual control** of offset commits using `commitSync()` and `commitAsync()`. 🚀
