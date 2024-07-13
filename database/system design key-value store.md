

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
