
## Kafka Manages Data and Metadata Separately

![[Pasted image 20230929231914.png]]

The functions within a Kafka cluster are broken up into a data plane and a control plane. 

- The control plane handles management of all the metadata in the cluster. 
-  The data plane deals with the actual data that we are writing to and reading from Kafka.

## Inside the Apache Kafka Broker

![[Pasted image 20230930092042.png]]

### Inside the Apache Kafka Broker

#### Partition Assignment

- configurable partitioner to determine the topic partition to assign to the record. 
- If the record has a key, then the default partitioner will use a hash of the key to determine the correct partition. Any records with the same key will always be assigned to the same partition. 
- If the record has no key then a partition strategy is used to balance the data in the partitions.

#### Record Batching

![[Pasted image 20230930113853.png]]

the producer will accumulate the records assigned to a given partition into batches.

![[Pasted image 20230930114005.png]]

The producer also has control as to when the record batch should be drained and sent to the broker.

One is by time. The other is by size.

#### Network Thread Adds Request to Queue

![[Pasted image 20230930115200.png]]

The request first lands in the broker’s socket receive buffer where it will be picked up by a network thread from the pool. That network thread will handle that particular client request through the rest of its lifecycle. The network thread will read the data from the socket buffer, form it into a produce request object, and add it to the request queue.

#### I/O Thread Verifies and Stores the Batch

![[Pasted image 20230930115551.png]]

----
##### Reference
https://developer.confluent.io/courses/architecture/broker/