
1. Commit Sync
2. Commit async

async commit은 commit 결과를 care 하지 않음.
Sync와 달리 retry도 하지 않음. 어차피 추후 뒤에 발생하는 커밋이 커밋되면 increment 되어서 이전 offset 위치가 커밋 될 필요가 없음. 커밋별로 block 되지 않음.
async commit을 쓰더라도, close and exit 전에는 sync commit을 보통 실행하도록 구현함. 

```java
try {
    consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Arrays.asList(topicName));

	while (true) {
		ConsumerRecords<String, Supplier> records = consumer.poll(100);
		for (ConsumerRecord<String, Supplier> record : records) {
			System.out.println("Supplier id= " + String.valueOf(record.value().getID()) +
					" Supplier  Name = " + record.value().getName() +
					" Supplier Start Date = " + record.value().getStartDate().toString());
		}
		consumer.commitAsync();
	}
} catch (Exception ex) {
	ex.printStackTrace();
} finally {
	consumer.commitSync();
	consumer.close();
}
```
