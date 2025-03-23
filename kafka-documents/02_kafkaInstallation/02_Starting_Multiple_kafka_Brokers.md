# Starting Multiple Brokers in Kafka

## **🛠 Step 1: Prepare Your Kafka Installation**

Ensure Kafka is installed and working correctly on your system. If you haven't installed Kafka yet, download and extract it as discussed in the previous section.

---

## **🔹 Step 2: Copy and Modify Broker Configurations**

Kafka uses a **single broker by default**. To run multiple brokers, you must create separate configurations for each.

### **1️⃣ Copy the default Kafka configuration file for each broker:**

```sh
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```

### **2️⃣ Modify each configuration file (`server-1.properties` and `server-2.properties`):**

#### **server-1.properties (for Broker 1):**

```ini
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/tmp/kafka-logs-1
```

#### **server-2.properties (for Broker 2):**

```ini
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs=/tmp/kafka-logs-2
```

### **Key Changes:**

- **`broker.id`** → Each broker must have a **unique ID**.
- **`listeners`** → Each broker must use a **different port** (e.g., `9093`, `9094`).
- **`log.dirs`** → Separate **log directories** for each broker.

---

## **🚀 Step 3: Start Multiple Brokers**

Now, start each broker with its respective configuration:

```sh
bin/kafka-server-start.sh config/server-1.properties &
bin/kafka-server-start.sh config/server-2.properties &
```

The `&` symbol runs the brokers in the background.

---

## **🔍 Step 4: Verify Running Brokers**

Check the brokers running in your cluster using:

```sh
bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9093
bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9094
```

You should see information about both brokers.

---

To check the brokers

```sh
kafka-broker-api-versions.sh --bootstrap-server localhost:9092
```
