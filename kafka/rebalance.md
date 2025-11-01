
https://www.confluent.io/blog/incremental-cooperative-rebalancing-in-kafka/

`scheduled.rebalance.max.delay.ms` default 5 min.

-> 5 분간 처리가 안되는 partition이 생기겠네?

-> 이 딜레이는 어디서 관리되나? group coordinator 겠지?
딜레이 카운팅 되는 상태는 그럼 다른 consumer들도 인지는 되는 상황일까?


---
	keep
https://cwiki.apache.org/confluence/display/KAFKA/KIP-429%3A+Kafka+Consumer+Incremental+Rebalance+Protocol

https://cwiki.apache.org/confluence/display/KAFKA/KIP-345%3A+Introduce+static+membership+protocol+to+reduce+consumer+rebalances