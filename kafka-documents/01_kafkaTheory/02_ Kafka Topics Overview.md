# Kafka Topics Overview

This document provides an introduction to **Kafka Topics**, including **partitions**, **offsets**, and other essential concepts. It is based on a lecture by Stephane from Conduktor.

---

## Table of Contents

- [Kafka Topics Overview](#kafka-topics-overview)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [What is a Kafka Topic?](#what-is-a-kafka-topic)
  - [Partitions](#partitions)
  - [Offsets](#offsets)
  - [Example: trucks\_gps Use Case](#example-trucks_gps-use-case)
  - [Key Points to Remember](#key-points-to-remember)
  - [Conclusion](#conclusion)

---

## Introduction

Kafka Topics are at the heart of Apache Kafka. A **topic** represents a **stream of data** within the Kafka cluster, and you can have as many topics as you need, each serving a different type of data (e.g., `logs`, `purchases`, `twitter_tweets`, `trucks_gps`, etc.).

---

## What is a Kafka Topic?

- A **topic** in Kafka is akin to a **table** in a database (but without constraints).
- You can send **any kind of data** (JSON, Avro, text, binary, etc.) to a topic.
- Topics are **identified by their names** (e.g., `logs`, `purchases`).
- Data is written to a topic by **Producers** and read from a topic by **Consumers**.
- There is **no querying** capability in Kafka itself (unlike databases).
- Sequence of messages is called a data stream.

---

## Partitions

- **Topics** are subdivided into **partitions**.
- Each partition is a **sequential** and **ordered** log of messages.
- A topic can have **1 or many** partitions (e.g., 3, 10, 100, etc.).
- **Order** of messages is guaranteed **only** within a single partition, **not** across partitions.

Example: If a topic has 3 partitions (`0`, `1`, `2`):

- Messages in partition 0 follow an order: offsets `0`, `1`, `2` ...
- Messages in partition 1 follow another order: offsets `0`, `1`, `2` ...
- Messages in partition 2 follow another order: offsets `0`, `1`, `2` ...

---

## Offsets

- Each message within a partition is assigned an **offset**, a unique sequential ID.
- Offsets are **per-partition** (i.e., two messages in different partitions can both have offset `3`, but they are not the same message).
- **Offsets** only increase; they are **never re-used**, even if older messages are deleted.
- Messages are **immutable**; once written, they cannot be updated or removed from a partition.  
  (Kafka does allow data retention settings, but the content itself cannot be altered in place.)

---

## Example: trucks_gps Use Case

Consider you have a fleet of trucks, each equipped with a GPS device:

1. **Each truck** sends its **location** (latitude, longitude) to a Kafka topic named `trucks_gps` every 20 seconds.
2. **Producers** (in this case, the trucks) write messages to the `trucks_gps` topic.
3. This `trucks_gps` topic might be configured to have **10 partitions**.
4. **Consumers** can then read from `trucks_gps` for different purposes:
   - A **location dashboard** service that displays real-time truck positions.
   - A **notification** service that alerts customers as deliveries get closer.

Because **multiple consumers** can read the same topic independently, Kafka enables **various services** to leverage the same data stream simultaneously.

---

## Key Points to Remember

1. **Immutability**: Messages are never changed once written.
2. **Retention**: Data is stored only for a **limited time** (default is **1 week**), though this is configurable.
3. **Offsets Are Partition-Specific**: Offset numbers are only meaningful within the context of a single partition.
4. **Ordering Guarantees**: Order is guaranteed **within** a partition, **not** across partitions.
5. **Random or Key-Based Partitioning**:
   - By default, messages are distributed to partitions in a **round-robin** or **random** fashion.
   - Providing a **key** ensures all messages with the same key go to the **same** partition.

---

## Conclusion

Kafka Topics, along with their **partitions** and **offsets**, form the foundation of how Apache Kafka manages streams of data. Knowing these basics helps you design scalable, real-time data architectures and allows different systems within your organization to **produce** and **consume** data in a **decoupled**, **efficient**, and **fault-tolerant** manner.
