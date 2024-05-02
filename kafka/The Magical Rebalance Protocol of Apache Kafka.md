
### Requirements

1. Partitions are automatically assigned when consumers join and leave groups.
2. Partition assignment must be decided by clients.
3. Support large number of groups
4. Support large group with few hundred consumers.
5. Consistency - one decision is made, all clients in group must comply

### Whats a protocol to do?

- Join Group
- Assign Resources, via a group leader.
- Maintain group membership

### Layerd protocol

- Kafka 
