# Kafka KRaft Mode

This document provides an overview of **Kafka KRaft mode**, a new architecture that eliminates the Zookeeper dependency from Apache Kafka.

---

## Table of Contents

- [Kafka KRaft Mode](#kafka-kraft-mode)
  - [Table of Contents](#table-of-contents)
  - [Background and Motivation](#background-and-motivation)
  - [Benefits of KRaft](#benefits-of-kraft)
  - [KRaft Implementation and Releases](#kraft-implementation-and-releases)
  - [KRaft vs. Zookeeper Architecture](#kraft-vs-zookeeper-architecture)
  - [Performance Improvements](#performance-improvements)
  - [Conclusion](#conclusion)

---

## Background and Motivation

- **KIP-500** (started around 2020) introduced the plan to **remove Zookeeper** from Kafka.
- **Scaling issues** emerged with Zookeeper-based Kafka clusters, especially with **100,000+ partitions**.
- The **goal**: allow **millions of partitions**, simplify **setup** and **maintenance**, and unify security.

---

## Benefits of KRaft

1. **Single Security Model**
   - Only **Kafka security** needs to be managed; no separate Zookeeper security.
2. **Simplified Operations**
   - Fewer components to monitor and support.
   - Only a single process to start (`kafka-server-start` in KRaft mode).
3. **Improved Stability**
   - Better handling of controller shutdowns and recoveries.
4. **Enhanced Scalability**
   - Potential to **support more partitions** with less operational overhead.

---

## KRaft Implementation and Releases

- **Kafka 3.x** introduced KRaft mode; initially **experimental**.
- **Production-ready** since **Kafka 3.3.1** (as part of **KIP-833**).
- **Kafka 4.0** will only support **KRaft** (no Zookeeper option).

---

## KRaft vs. Zookeeper Architecture

- **Zookeeper-Based**
  - Requires a **Zookeeper Quorum** (leader + followers) plus **Kafka brokers**.
  - Zookeeper manages **broker metadata** and **leader election**.
- **KRaft-Based**
  - Uses an internal **Quorum Controller** (part of the **Kafka broker** itself).
  - One broker acts as the **Quorum Leader**, handling **metadata** and **leader elections** directly.
  - No external Zookeeper process needed.

---

## Performance Improvements

- **Faster Controller Shutdown**
  - KRaft reduces the time needed for a clean shutdown of the controller.
- **Quicker Recovery**
  - After an unplanned outage, **recovery time** is notably shorter.
- Overall, these improvements lead to **better availability** and **reduced downtime**.

---

## Conclusion

Kafka KRaft mode represents the **future** of Apache Kafka:

- Streamlined deployment (no Zookeeper dependency).
- Improved **performance**, **stability**, and **scalability**.
- Already **production-ready** in Kafka **3.3.1** and beyond, and will be the **only** option in **Kafka 4.0**.
