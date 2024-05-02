
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

크게 세개로 나눠볼 수 있음

- Kafka Protocol: Group membership
	- it's responsible for who is a member in this group and who is a leader and then there is the group protocol itself
- Group Protocol: Resource definition, assignment process.
	- its protocol that members of the client consumer group used to basically decide on a location how do they communicate with each other how do remember I said that all the information exists on the client but it exists on the clients in a collective way
	- we needed the leader who makes the decision to have all this collective information you need a language for that 
- Sub-protocol: Assignment algorithm + "user data"

