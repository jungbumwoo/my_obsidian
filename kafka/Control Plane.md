Manage meta data of the cluster.


###  Metadata
- In KRaft mode, cluster metadata, reflecting the current state of all controller managed resources, is stored in a single partition Kafka topic called cluster_metadata. __cluster_metadata 으로 metadata를 관리함

-  The active controller is the leader of the metadata topic’s single partition and it will receive all writes. The other controllers are followers and will fetch those changes.

- 일반 data plane topic 이랑 차이점:
	- when a leader needs to be elected, this is done via quorum, rather than an in-sync replica set. So, there is no ISR involved in metadata replication. 
	- Another difference is that metadata records are flushed to disk immediately as they are written to each node’s local log.

### Leader Election

#### #### Vote Request[](https://developer.confluent.io/courses/architecture/control-plane/#vote-request)

다른 controller 들에 VoteRequest를 보냄. lastOffset, lastOffsetEpoch, candidateEpoch 보냄

#### Vote Response
1. If higher epoch is seen, reject.
2. If already Vote on epoch, send the same answer
3. 

https://developer.confluent.io/courses/architecture/control-plane/