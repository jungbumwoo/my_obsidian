

### Producer
#### Producer restart
When producer starts, if the idempotent producer is enabled, the prodcuer will initialize and reach out a Kafka broker to generate a producer ID. 

1.
If a prodcuer fails and the producer that replaces it sends a message that was previously sent by the old producer, the broker will not detect the duplicates.

2.
If the old producer froze and then came back to life after its replacement started, the original producer is not recognized as a zombie.

idempotent producer only prevents duplicates in case of retries that are caused by the producer's internal logic.

#### Broker failure

fellower 가 leader로 승격되더라도 producer with the five last sequence id  까지는 동기화하고 있음.

what happends when the old leader comes backs?
snapshot , log 이용해서 follow up .


### What Problems Do Transactions solve?

reads events from source topic, maybe process them, and writes result to another topic

#### Reprocessing caused by application crash
A 토픽에 consume 후 B 토픽에 produce 되었으나 A consume 을 커밋하지 못한 경우

#### Reprocessing caused by zombie applications
consume 해놓고 heart beat miss 로 파티션이 다른 consumer에 할당된 상태에서 둘다 consume msg가 processing 됨


https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals

https://github.com/apache/kafka/pull/7115


### Consumer

Reprocessing caused by application crashes
Reprocessing caused by zombie applications

https://cwiki.apache.org/confluence/display/KAFKA/KIP-447%3A+Producer+scalability+for+exactly+once+semantics

https://docs.google.com/document/d/1LhzHGeX7_Lay4xvrEXxfciuDWATjpUXQhrEIkph9qRE/edit?tab=t.0#heading=h.beexgkt7kkor

https://docs.google.com/document/d/11Jqy_GjUGtdXJK94XGsEIK7CP1SnQGdp2eF0wSw9ra8/edit?tab=t.0#heading=h.xq0ee1vnpz4o

아직 이해 안간 부분
To preserve the static partition mapping in a consumer group where assignments are frequently changing, the simplest solution is to create a separate producer for every input partition. This is what Kafka Streams does today. 
https://cwiki.apache.org/confluence/display/KAFKA/KIP-447%3A+Producer+scalability+for+exactly+once+semantics

https://www.confluent.io/blog/apache-kafka-2-5-latest-version-updates/