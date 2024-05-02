
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
	- 컨슈머 간 커뮤니케이션.
	- all the information exists on client in a collective way.
	- wee need the leader who makes the decision to have all this collective information you need a language for that and this is the group protocol.
- Sub-protocol: Assignment algorithm + "user data"
	- how do we assign partitions.

