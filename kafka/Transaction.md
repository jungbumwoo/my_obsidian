

트랜잭션과 별개로.. 문득 드는 생각, 오토커밋을 하지 않는 상황. 특정 msg 처리 과정에서 에러가 발생하면 msg부터 정상 처리가 안되는 경우, 해결 될 때 까지 해당 특정 파티션의 이후 msg들이 처리가 안되는 케이스가 발생할듯


### Transactional ID
until 2.5, the only way to guarantee  fencing was to statically map the transactional ID to partitions.
In Apache Kafka 2.5, KIP-447, 
consumer group metadata for fencing in addition to transactional IDs.

트랜잭션은 Kafka 내부에서 데이터 쓰기 작업에 정확히 한 번만 처리를 보장하지만, 읽기나 외부 시스템과의 연동에는 별도의 작업이 필요함

### How Do Transactions Guarantee Exactly-Once?



**transaction coordinator**
manage transactions of messages sent by producers, and commit / abort the appends of these messages as a whole



https://docs.google.com/document/d/11Jqy_GjUGtdXJK94XGsEIK7CP1SnQGdp2eF0wSw9ra8/edit?tab=t.0


### Transactional Messaging In Kafka

https://cwiki.apache.org/confluence/display/KAFKA/Transactional+Messaging+in+Kafka