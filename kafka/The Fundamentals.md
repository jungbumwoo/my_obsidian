
Kafka is a data streaming system that allows developers to react to new events as they occur in real time. Kafka architecture consists of a storage layer and a compute layer. The storage layer is designed to store data efficiently and is a distributed system such that if your storage needs grow over time you can easily scale out the system to accommodate the growth. The compute layer consists of four core components—the producer, consumer, streams, and connector APIs, which allow Kafka to scale applications across distributed systems.


## What Is an Event in Stream Processing?

![[Pasted image 20230928235549.png]]

An event record consists of a timestamp, a key, a value, and optional headers. The event payload is usually stored in the value. The key is also optional, but very helpful for event ordering, colocating events across topics, and key-based storage or compaction.

## Record Schema

![[Pasted image 20230928235803.png]]

## Kafka Topics

Topics are append-only, immutable logs of events

### Kafka Topic Partitions

![[Pasted image 20230929000033.png]]



next: [[Inside the Apache Kafka Broker]]
