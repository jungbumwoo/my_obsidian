
### **Consistency**
Every read receives the most recent write or an error.

-  all clients see the same data at the same time, no matter which node they connect to
-  For this to happen, whenever data is written to one node, it must be instantly forwarded or replicated to all the other nodes in the system before the write is deemed ‘successful.’

### **Availability**
Every request receives a (non-error) response, without the guarantee that it contains the most recent write.

- Availability means that any client making a request for data gets a response, even if one or more nodes are down.
- Another way to state this—all working nodes in the distributed system return a valid response for any request, without exception.

### **Partition tolerance**
- A partition is a communications break within a distributed system—a lost or temporarily delayed connection between two nodes. Partition tolerance means that the cluster must continue to work despite any number of communication breakdowns between nodes in the system.

When a [network partition](https://en.wikipedia.org/wiki/Network_partition "Network partition") failure happens, it must be decided whether to do one of the following:

- cancel the operation and thus decrease the availability but ensure consistency
- proceed with the operation and thus provide availability but risk inconsistency.


CAP Twelve Years Later: How the "Rules" Have Changed
https://ieeexplore.ieee.org/document/6133253


http://eincs.com/2013/07/misleading-and-truth-of-cap-theorem/