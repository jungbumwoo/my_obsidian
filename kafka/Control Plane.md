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
3. Otherwire, grant vot only if the candidate's log is at least as "long"

#### #### Completion

the new leader will send a BeginQuorumEpoch request, including the new epoch, to the other controllers

#### KRaft Cluster Metadata Snapshot[](https://developer.confluent.io/courses/architecture/control-plane/#kraft-cluster-metadata-snapshot)

metadata가 log 로 계속 쌓이는건 아니고 snapshot을 offset, epoch 랑 함께 남김

주로 broker가 restart, new brokers coming online 시 snapshot을 이용함

developer.confluent.io/courses/architecture/control-plane/