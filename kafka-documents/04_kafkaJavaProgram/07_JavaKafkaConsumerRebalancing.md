# Kafka Consumer Groups and Rebalancing - Java Tutorial Notes

This document summarizes how Kafka consumer groups work and how **partition rebalancing** occurs when multiple consumers join or leave the same group.

---

## 🎯 Objective

- Understand **Kafka consumer groups**
- Learn about **partition assignment and rebalancing**
- Observe consumer behavior with **multiple instances**

---

## 🧪 Scenario Setup

- Topic has **3 partitions**
- Multiple instances of the same Java consumer app (`ConsumerDemoWithShutdown`) are launched
- All consumers share the same **group ID** (`my-java-application`)

---

## 🧱 Initial Consumer

- Run the application with **1 consumer**
- Joins the group and gets assigned **all 3 partitions**
- Since `auto.offset.reset=latest`, it starts consuming only new messages

> Log example: Group is stable, lag is 3

---

## ➕ Adding More Consumers

### 1. **Run a Second Instance**

- Modify configuration to allow **multiple instances**
- Launch second instance of `ConsumerDemoWithShutdown`

> 📌 Observation:
>
> - Logs show that **group is rebalancing**
> - Partitions are reassigned:
>   - First consumer gets `demo_java-0` and `demo_java-1`
>   - Second consumer gets `demo_java-2`

### 2. **Start the Producer**

- Run `ProducerDemoWithKeys` to send messages to all partitions
- Logs show:
  - Consumer 1 reads partitions 0 and 1
  - Consumer 2 reads partition 2

✅ Partition assignment is working correctly.

---

## ➕ Add a Third Consumer

- Run a **third instance** of `ConsumerDemoWithShutdown`

> 📌 Observation:
>
> - Now each consumer gets **1 partition**:
>   - Consumer 1: `demo_java-0`
>   - Consumer 2: `demo_java-2`
>   - Consumer 3: `demo_java-1`

---

## 🧼 Demonstrate Clean Shutdown

### Step 1: Shutdown Consumer 3

- Other consumers rebalance
- Now:
  - Consumer 1: `demo_java-0` and `demo_java-1`
  - Consumer 2: `demo_java-2`

### Step 2: Shutdown Consumer 2

- Final remaining consumer gets **all 3 partitions**

---

#### Logs:

```
Notifying assignor about the new Assignment(partitions=[demo_java-2])
Adding newly assigned partitions: demo_java-2
Setting offset for partition demo_java-2
```

---

## Kafka Consumer Group Rebalancing When Consumers > Partitions

What happens when a Kafka consumer group has **more consumers than partitions**. It builds on the previous example with a topic that has **3 partitions** and explores the behavior when you increase the number of consumers beyond that.

---

## 🧠 Kafka Basics Refresher

- A **partition** is the unit of parallelism in Kafka.
- A **consumer group** is a set of consumers that cooperate to consume data from a topic.
- Each **partition is assigned to only one consumer** in a consumer group.
- A **consumer can read from multiple partitions**, but **a partition cannot be read by multiple consumers**.

---

## 🧪 Scenario Setup

- Topic: `demo_java`
- Partition count: `3`
- Start with 3 consumers → then add a 4th

---

## ⚙️ Behavior With 3 Consumers

Each partition gets assigned to exactly one consumer:

| Consumer | Assigned Partition |
| -------- | ------------------ |
| C1       | demo_java-0        |
| C2       | demo_java-1        |
| C3       | demo_java-2        |

✅ **All consumers are active and utilized**

---

## ➕ Adding a 4th Consumer

When a 4th consumer (`C4`) joins the group:

> Kafka **rebalances** the group, but only 3 partitions are available.

So, the outcome is:

| Consumer | Assigned Partition           |
| -------- | ---------------------------- |
| C1       | demo_java-0                  |
| C2       | demo_java-1                  |
| C3       | demo_java-2                  |
| C4       | **No partition assigned** ❌ |

⚠️ **C4 becomes idle** — it joins the group but receives no data.

---

## 🤔 Why Doesn’t Kafka Split Partitions?

Kafka ensures **message order within a partition**, so it cannot split a partition across multiple consumers. Therefore:

- If you have more consumers than partitions, the **extra consumers remain idle**
- Adding more consumers than partitions does **not** improve throughput

---

## 🧼 Rebalancing Flow Recap

1. A new consumer joins
2. Kafka **revokes** current partition assignments
3. Kafka **redistributes** partitions evenly (1-to-1)
4. **Excess consumers** are left idle

This automatic behavior ensures message processing consistency.

---

## 📝 Summary

| Partitions | Consumers | Result                            |
| ---------- | --------- | --------------------------------- |
| 3          | 1         | 1 consumer handles all partitions |
| 3          | 3         | Perfect 1:1 mapping               |
| 3          | 4+        | Extra consumers are idle          |

📌 **Tip**: For optimal utilization, try to keep the number of consumers **less than or equal** to the number of partitions.

---

## ✅ Best Practice

If you need more parallelism, **increase the number of partitions** rather than just adding consumers.

---

Now you understand how Kafka handles consumers in excess of partitions! This is a key concept when designing scalable consumer groups. 🚀

## ✅ Summary

| Consumers       | Partition Assignment   |
| --------------- | ---------------------- |
| 1               | 0, 1, 2                |
| 2               | 0,1 (C1), 2 (C2)       |
| 3               | 0 (C1), 1 (C3), 2 (C2) |
| After shutdowns | All to last consumer   |

Kafka handles rebalance automatically to evenly distribute partitions across available consumers in a group.

---

## 💡 Key Takeaways

- **Group ID** links consumers into a consumer group
- Kafka auto-assigns partitions to group members
- On joining/leaving, Kafka **rebalances** partitions
- Rebalancing ensures efficient message processing

---

This foundational knowledge is critical for scaling Kafka consumers and will lead into deeper discussions on partition rebalancing strategies. See you in the next lecture! 🚀
