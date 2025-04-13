# Kafka Consumer With Graceful Shutdown - Java Tutorial Notes

This document explains how to implement a **graceful shutdown** for a Kafka consumer using Java. The process involves creating a shutdown hook, using `consumer.wakeup()`, and handling exceptions to close the consumer cleanly.

---

## 🎯 Objective

- Add a **shutdown hook** to gracefully terminate a Kafka consumer
- Ensure proper **cleanup**, **offset commit**, and **partition revocation**

---

## 🧱 Setup Steps

### 1. Duplicate Consumer Code

- Start by duplicating the `ConsumerDemo` and rename it to:

```java
ConsumerDemoWithShutdown
```

---

### 2. Keep Existing Consumer Setup

Keep all initial configurations:

- Properties
- Group ID
- Deserializers

```java
Properties properties = new Properties();
// set bootstrap.servers, key/value deserializers, group.id, and auto.offset.reset
```

---

## 🔄 Add Shutdown Hook

### Step 1: Reference Main Thread

```java
final Thread mainThread = Thread.currentThread();
```

### Step 2: Register the Shutdown Hook

```java
Runtime.getRuntime().addShutdownHook(new Thread() {
    public void run() {
        log.info("Detected a shutdown, calling consumer.wakeup()");
        consumer.wakeup();
        try {
            mainThread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});
```

### 🔍 Why this works:

- `consumer.wakeup()` triggers a `WakeupException` during the next `poll()`
- `mainThread.join()` ensures the hook waits for main thread code to complete

---

## 🧪 Wrap Consumer Logic with Try-Catch-Finally

### Example Structure:

```java
try {
    consumer.subscribe(Arrays.asList("demo_java"));

    while (true) {
        log.info("Polling");
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));

        for (ConsumerRecord<String, String> record : records) {
            log.info("Key: {} | Value: {}", record.key(), record.value());
            log.info("Partition: {} | Offset: {}", record.partition(), record.offset());
        }
    }
} catch (WakeupException e) {
    log.info("Consumer is starting to shut down");
} catch (Exception e) {
    log.error("Unexpected exception in the consumer", e);
} finally {
    consumer.close();
    log.info("The consumer is now gracefully shut down");
}
```

---

## ✅ What Happens During Graceful Shutdown

- when signals—like Ctrl+C in a terminal—or if the JVM receives a normal SIGTERM rather than a forced kill. `consumer.wakeup()` triggers `WakeupException`
- Exits the polling loop
- `consumer.close()` is called:
  - Commits offsets
  - Leaves the group
  - Revokes partitions
  - Shuts down internal metrics
- Note : When you click Terminate in Eclipse, it generally kills the JVM immediately (effectively a “hard kill”), so the JVM never runs the shutdown hook that would produce your log messages. The shutdown hook only triggers on “graceful shutdown” signals—like Ctrl+C in a terminal—or if the JVM receives a normal SIGTERM rather than a forced kill.

---

## 📦 Output Logs (Sample)

- Thread detects shutdown:

```bash
[Thread-0] Detected a shutdown, calling consumer.wakeup()
```

- Wakeup exception caught:

```bash
[main] Consumer is starting to shut down
```

- Kafka logs:

```bash
Revoking previously assigned partitions...
Resetting generation...
Leaving the group...
```

- Final shutdown log:

```bash
The consumer is now gracefully shut down
```

---

## 📝 Summary

| Step                | Description                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| `mainThread`        | Save reference to main thread                                                   |
| `shutdownHook`      | Add hook to call `consumer.wakeup()`                                            |
| `try-catch-finally` | Handle `WakeupException`, unknown exceptions, and always `close()` the consumer |

This approach allows the consumer to exit cleanly, commit offsets, and prepare for the next use case: **Consumer Groups**.

---

Happy consuming, see you in the next Kafka module! 🚀
