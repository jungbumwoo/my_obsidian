Data plane : how to client request and replication is handle. 

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

- The leader will respond to the fetch request with the records starting at the specified offset. 
- current epoch is fetched too.

### Committing Partition Offsets

![[Pasted image 20231105225205.png]]

- leader manage high watermark by fetch request offsets

### Advancing the Follower High Watermark

![[Pasted image 20231105231958.png]]

- The leader, in turn, uses the fetch response to inform followers of the current high watermark. 
- Because this process is asynchronous, **the followers’ high watermark will typically lag behind the actual high watermark** held by the leader.

### Handling Leader Failure
![[Pasted image 20231105233619.png]]

### Temporary Decreased High Watermark
![[Pasted image 20231105233654.png]]

### Partition Replica Reconciliation

![[Pasted image 20231105233544.png]]

All of the controller brokers maintain an in-memory metadata cache that is kept up to date, so that any controller can take over as the active controller if needed.


### KRaft Cluster Metadata




------

prev: [[Inside the Apache Kafka Broker]]

