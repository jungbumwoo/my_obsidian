## Control Plane
![[Pasted image 20231108231010.png]]

### KRaft Cluster Node Roles

![[Pasted image 20231108231107.png]]

 Which way to go will depend on the size of your cluster.

### KRaft Mode Controller

![[Pasted image 20231108231158.png]]


### KRaft Cluster Metadata

![[Pasted image 20231108232034.png]]

In KRaft mode, cluster metadata, reflecting the current state of all controller managed resources, is stored in a single partition Kafka topic called __cluster_metadata.

### KRaft Metadata Replication

![[Pasted image 20231108232222.png]]

- The active controller is the leader of the metadata topic’s single partition and it will **receive all writes**.
- However, when a leader needs to be elected, this is done via quorum, rather than an in-sync replica set. So, there is no ISR involved in metadata replication. 
- Another difference is that metadata **records are flushed to disk immediately** as they are written to each node’s local log.