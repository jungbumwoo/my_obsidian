


### Transactional ID
until 2.5, the only way to guarantee  fencing was to statically map the transactional ID to partitions.
In Apache Kafka 2.5, KIP-447, 
consumer group metadata for fencing in addition to transactional IDs.

트랜잭션은 Kafka 내부에서 데이터 쓰기 작업에 정확히 한 번만 처리를 보장하지만, 읽기나 외부 시스템과의 연동에는 별도의 작업이 필요함

### How Do Transactions Guarantee Exactly-Once?



**transaction coordinator**
manage transactions of messages sent by producers, and commit / abort the appends of these messages as a whole



https://docs.google.com/document/d/11Jqy_GjUGtdXJK94XGsEIK7CP1SnQGdp2eF0wSw9ra8/edit?tab=t.0
