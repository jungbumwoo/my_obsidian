## Kafka Data Replication

![[Pasted image 20231105162116.png]]

When a new topic is created we can specify, explicitly or through defaults, how many replicas we want. Then each partition of that topic will be replicated that many times.

### Leader, Follower, and In-Sync Replica (ISR) Lis

![[Pasted image 20231105162452.png]]

The partition leader, along with all of the followers that have caught up with the leader, will be part of the in-sync replica set (ISR). In the ideal situation, all of the replicas will be part of the ISR.


### Leader Epoch
![[Pasted image 20231105170115.png]]


### Follower Fetch Request

![[Pasted image 20231105170539.png]]

Whenever the leader appends new data into its local log, the followers will issue a fetch request to the leader, passing in the offset at which they need to begin fetching.

### Follower Fetch Response

![[Pasted image 20231105170620.png]]

The leader will respond to the fetch request with the records starting at the specified offset. The fetch response will also include the offset for each record and the current leader epoch. The followers will then append those records to their own local logs.

----

prev: [[Inside the Apache Kafka Broker]]

