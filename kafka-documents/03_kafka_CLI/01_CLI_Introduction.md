# Kafka CLI Introduction

## Introduction

This section covers the **Kafka CLI** and how to use it effectively. The Kafka CLI is bundled with the Kafka binaries, and if your **path variable** is set up correctly, you can invoke the CLI commands from anywhere on your computer.

## Running Kafka CLI Commands

To check if Kafka CLI is working, try running:

```sh
kafka-topics
```

If installed correctly, it should return the command results.

### OS-Specific Commands

Depending on your OS, you may need to use different suffixes for CLI commands:

- **Linux / Mac**: `kafka-topics.sh`
- **Windows WSL2**: `kafka-topics.sh`
- **Windows (non-WSL2)**: `kafka-topics.bat`
- **Homebrew / apt installations**: `kafka-topics`

Ensure that you use the correct command format for your operating system.

## Using Bootstrap Server Instead of Zookeeper

Kafka now uses `--bootstrap-server` instead of `--zookeeper` since **Zookeeper is being phased out**.

Starting from **Kafka 3.0**, use the following format:

```sh
kafka-topics.sh --bootstrap-server localhost:9092
```

not

```sh
kafka-topics.sh --zookeeper localhost:2181
```

instead of referencing Zookeeper, even though Zookeeper might still be available.

## Troubleshooting CLI Issues

If you encounter issues with Kafka CLI, it could be due to an incorrect **path setup**. If setting the path doesn't work, use the **full path** to the Kafka binaries:

```sh
/path/to/kafka/bin/kafka-topics.sh
```

If running `kafka-topics` results in a **command not found** error, locate your Kafka directory, such as:

```
kafka_2.13-3.1.0/bin/
```

Then, execute the command directly from the **bin** directory:

```sh
./kafka-topics.sh
```

This should resolve any issues.

## Summary

If your path is not set up correctly:

1. Revisit the setup steps and configure the path correctly.
2. If the issue persists, use the **full path** to the Kafka binaries.

Once these steps are complete, you should be ready to use the Kafka CLI effectively.
