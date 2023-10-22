
## AdminClient Overview

### Asynchronous and Eventually Consistent API

- Kafka’s AdminClient is that it is asynchronous.
- Each method returns immediately after delivering a request to the cluster controller, and each method returns one or more Future objects.
- Future objects are the result of asynchronous operations, and they have methods for check‐ ing the status of the asynchronous operation, canceling it, waiting for it to complete, and executing functions after its completion.


### Modifying Consumer Group

- consumer groups don’t receive updates when offsets change in the offset topic. They only read offsets when a consumer is assigned a new partition or on startup. 
- To prevent you from making changes to offsets that the consumers will not know about (and will therefore override), Kafka will prevent you from modifying offsets while the consumer group is active.