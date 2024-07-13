
### CAP Theorem
It is impossible for a distributed system to simultaneously provide more than two of these three guarantees.

- Consistency:  all clients see the same data at the same time.
- Availability: any client which requests data gets a response even if some of the nodes are down.
- Partition Tolerance: a partition indicates a communication break between two nodes. Partition tolerance means the system continues to operate despite network partitions.

Since network failure is unavoidable, a distributed system must tolerate network partition.

### Data partition
There are two challenges while partitioning the data:
- Distribute data across multiple servers evenly.
- Minimize data movement when nodes are added or removed.

### Data replication
After a key is mapped to a position on the hash ring, walk clockwise from that position and choose the first unique N servers .

### Consistency
N = number of replica
W = A write quorum of size W.
R = A read quorum of size R.

The configuration of W, R and N is a typical tradeoff between latency and consistency.

### Consistency models

- strong consistency: any read operation returns a value corresponding to the result of the most updated write data item.
- weak consistency: subsequent read operations may not see the most updated value.
- eventual consistency: this is a specific form of weak consistency. Given enough time, all updates are propagated, and all replicas are consistent.