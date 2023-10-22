
## AdminClient Overview

### Asynchronous and Eventually Consistent API

- Kafka’s AdminClient is that it is asynchronous.
- Each method returns immediately after delivering a request to the cluster controller, and each method returns one or more Future objects.
- Future objects are the result of asynchronous operations, and they have methods for check‐ ing the status of the asynchronous operation, canceling it, waiting for it to complete, and executing functions after its completion.


### Modifying Consumer Group

- consumer groups don’t receive updates when offsets change in the offset topic. They only read offsets when a consumer is assigned a new partition or on startup. 
- To prevent you from making changes to offsets that the consumers will not know about (and will therefore override), Kafka will prevent you from modifying offsets while the consumer group is active.

### Deleting Records from a Topic

- A topic with a retention policy of 30 days can store older data if all the data fits into a single segment in each partition.
- The deleteRecords method will mark as deleted all the records with offsets older than those specified when calling the method and make them inaccessible by Kafka consumers.

기한 설정이 되어 있다고 해서 바로 이전 데이터가 지워지는게 아니라ㅣ s