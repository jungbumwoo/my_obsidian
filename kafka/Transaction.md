


### Transactional ID
until 2.5, the only way to guarantee  fencing was to statically map the transactional ID to partitions.


트랜잭션은 Kafka 내부에서 데이터 쓰기 작업에 정확히 한 번만 처리를 보장하지만, 읽기나 외부 시스템과의 연동에는 별도의 작업이 필요함

### How Do Transactions Guarantee Exactly-Once?


