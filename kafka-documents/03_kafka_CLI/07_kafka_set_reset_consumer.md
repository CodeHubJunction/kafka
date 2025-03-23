# Kafka Offset Reset Guide

## **Introduction**

In this guide, you’ll learn how to **reset consumer offsets** using the Kafka CLI. This is especially useful when you want a consumer group to reprocess data from a Kafka topic — for example, from the beginning of the topic.

Kafka consumers track their progress using **offsets**. When offsets are committed, a consumer can resume reading from where it left off. But sometimes, you may want to reset those offsets.

---

## **1. View Current Offset and Lag**

Use this command to inspect a consumer group's lag:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
```

Example output:

```
TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
third-topic   0          14              16              2
third-topic   1          13              15              2
third-topic   2          25              26              1
```

This shows that **5 messages are waiting** to be consumed.

---

## **2. Dry Run an Offset Reset**

**Note: Refer Important Notes** 

To preview what a reset would do, run:

```sh
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-first-application \
  --topic third-topic \
  --reset-offsets --to-earliest --dry-run
```

### Output:

```
PARTITION  CURRENT-OFFSET  NEW-OFFSET
0          14              0
1          13              0
2          25              0
```

This means that all partitions would be reset to the **start of the topic**.

---

## **3. Execute Offset Reset**

To actually reset the offsets:

```sh
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-first-application \
  --topic third-topic \
  --reset-offsets --to-earliest --execute
```

Now, all offsets for the group `my-first-application` are reset to `0`.

---

## **4. Consume Messages After Reset**

Start a console consumer again:

```sh
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic third-topic \
  --group my-first-application
```

Because the offsets were reset, **all previous messages** in the topic will be consumed again.

Re-check the lag:

```sh
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
```

You should now see **lag = 0** across all partitions.

---

## **5. Important Notes**

- ❗ **You must stop all consumers** in the group before resetting offsets. If any are running, the reset won’t work.
- ✅ You can also reset offsets using a UI like **Conduktor**, where you can:
  - Select the topic and partition
  - Choose reset strategy (`earliest`, `latest`, `timestamp`, etc.)
  - Execute the reset with visual confirmation

---

## **6. Advanced Reset Options**

Kafka's CLI supports advanced options:

- `--to-latest` → Resets to the most recent offset
- `--to-offset <value>` → Resets to a specific offset
- `--to-datetime <timestamp>` → Resets to messages published after a specific time
- `--shift-by <N>` → Shift offset by N messages forward or backward
- `--to-current` → Resets to current committed offset (no-op in practice)

These are useful in debugging or replay scenarios, but they can get complex.

---

## **Conclusion**

You’ve learned how to:

- Inspect consumer group lag
- Preview and execute offset resets
- Consume reprocessed data
- Use both CLI and UI to manage offsets

Offset management is a powerful Kafka feature that lets you **replay data, recover from issues, or tune consumption logic**. See you in the next lecture! 🚀
