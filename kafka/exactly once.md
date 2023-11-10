
# Transactions in Apache Kafka

Transactions do three important things:

- They allow groups of messages to be sent, atomically, to different topics—for example, Order Confirmed and Decrease Stock Level, which would leave the system in an inconsistent state if only one of the two succeeded.
- They remove duplicates, which cause many streaming operations to get incorrect results (even something as simple as a count).
- [Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html?session_ref=https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/) and the [streaming database ksqlDB](https://ksqldb.io/) use state stores and state stores are backed by Kafka topics. For exactly-once stateful processing, multiple actions must happen atomically: writing to the state store, writing to the backing topic, writing the output, and committing offsets. Transactions enable this atomicity.


## **Failures that must be handled**

**A broker can fail**

Kafka can tolerate _n-1_ broker failures, meaning that a partition is available as long as there is at least one broker available.


1 .**The producer-to-broker RPC can fail**
2. **The producer-to-broker RPC can fail**
-  Since there is no way for the producer to know the nature of the failure, it is forced to assume that the message was not written successfully and to retry it. In some cases, this will cause the same message to be duplicated in the Kafka partition log, causing the end consumer to receive it more than once.
3. **The client can fail**