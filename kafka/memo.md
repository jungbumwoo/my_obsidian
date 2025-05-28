
connector 중단되고 다시 실행시점까지 bin log가 어디까지 보장되는건지?

snapshot 중간중간 계속 떠놓고 있어서 bin log가 아에 전부 필요한건 아닐듯.

  

  

Kafka Connect provides excellent fault tolerance and scalability, since it runs as a distributed service and ensures that all registered and configured connectors are always running. For example, even if one of the Kafka Connect endpoints in a cluster goes down, the remaining Kafka Connect endpoints will restart any connectors that were previously running on the now-terminated endpoint, minimizing downtime and eliminating administrative activities.

  

  

  

  

**multi target, multi source**

[https://debezium.io/blog/2018/01/17/streaming-to-elasticsearch/](https://debezium.io/blog/2018/01/17/streaming-to-elasticsearch/)

https://debezium.io/documentation/reference/2.7/transformations/event-flattening.html

  

  

빠르게

  

[_poll.interval.ms_](http://poll.interval.ms) 줄이기

  

_Mysql, MongoDB_ 와 같은 상이한 _DB_에서 동일 _topic_으로 쌓이는 상황도 있는건가_?_

  

  

**idempotent**

  

  

_https://debezium.io/blog/2020/02/25/lessons-learned-running-debezium-with-postgresql-on-rds/_

Each change record in PostgreSQL has a position which is tracked using a value known as a log sequence number (LSN). PostgreSQL represents it as two hexadecimal numbers - logical **xLog** and **segment**. 

  

  

  

[_https://www.conduktor.io/kafka/idempotent-kafka-producer/_](https://www.conduktor.io/kafka/idempotent-kafka-producer/)

어라_?_ 이러면 멱등성 보장하려고 _GO-back-N_ 방식으로 동작하는건지_?_ 

Selective repeat 방식이 아니라면 steaming performance 와 trade-off 가 있어보임

  

https://developer.confluent.io/patterns/event-processing/idempotent-writer/

Enabling idempotency for a Kafka producer not only ensures that duplicate Events are fenced out from the topic, it also ensures that they are written in order. 

  

[_https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging_](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging)

동일 partition 내에서만 sequence id 가 보장되는 것 아닌가?

  

producer가 새로 뜨면 (PID, seq) 에서 PID 변경되면서 seq 가 동일한게 먹힐 수 있는건지?ㅠ

  

  

_https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging_

  

When idempotence is enabled, we enforce that acks=all, retries > 1, and max.inflight.requests.per.connection=1. 

Without these values for these configurations, we cannot guarantee idempotence.

  

 _acks_ 는 반드시 _all_ 이어야만함

 _retries_는 _0_보다 커야한다_._

  

  

[_https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/_](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)

- exactly-once semantics per partition

  

  

  

  

  

- 딜레이를 최소화 하는 대용량 트래픽/데이터 전송 방안 

  

**Partition Routing**

  

By default, when Debezium detects a change in a data collection, the change event that it emits is sent to a topic that uses a single Apache Kafka partition. As described in [Customization of Kafka Connect automatic topic creation](https://debezium.io/documentation/reference/2.7/configuration/topic-auto-create-config.html#customizing-debezium-automatically-created-topics), you can customize the default configuration to route events to multiple partitions, based on a hash of the primary key.

However, in some cases, you might also want Debezium to route events to a specific topic partition. The partition routing SMT enables you to route events to specific destination partitions based on the values of one or more specified payload fields. To calculate the destination partition, Debezium uses a hash of the specified field values.

  

  

  

_FalutTorlerance_

  

_https://debezium.io/blog/2017/02/22/Debezium-at-WePay/_

MySQL prior to version 5.6 modeled a replica’s location in its parent’s binlogs using a (binlog filename, file offset) tuple. The problem with this approach is that the binlog filenames are not the same between MySQL machines

Starting with MySQL 5.6, MySQL introduced the concept of global transaction IDs.

  

  

_idempotent_ 특성을 이용하여_, failed_이 발생한 이전 시점부터 다시 _process_  를  진행하여 _failed_ 로부터 _resilience_를 갖을 수 있음

  

  

  

  

읽어보지 못한 글들

  

_Consistency at scale_

_https://www.uber.com/en-KR/blog/money-scale-strong-data/?uclick_id=87a4dbb9-3a79-4568-82ec-13e4f9a0f484_

  

Disaster Recovery for Multi-Region Kafka at Uber

https://www.uber.com/en-KR/blog/kafka/?uclick_id=87a4dbb9-3a79-4568-82ec-13e4f9a0f484