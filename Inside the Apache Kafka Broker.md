
## Kafka Manages Data and Metadata Separately

![[Pasted image 20230929231914.png]]

The functions within a Kafka cluster are broken up into a data plane and a control plane. 

- The control plane handles management of all the metadata in the cluster. 
-  The data plane deals with the actual data that we are writing to and reading from Kafka.

## Inside the Apache Kafka Broker

![[Pasted image 20230930092042.png]]

### Inside the Apache Kafka Broker

#### Partition Assignment

----
##### Reference
https://developer.confluent.io/courses/architecture/broker/