
https://cwiki.apache.org/confluence/display/KAFKA/Idempotent+Producer

There are two common reasons duplicate messages may occur:

1. If a client attempts to send a message to the cluster and gets a network error then retrying will potentially lead to duplicates. If network error occurred before the message was delivered, no duplication will occur. However if the network error occurs after the message is appended to the log but before the response can be delivered to the sender the sender is left not knowing what has happened. The only choices are to retry and risk duplication or to give up and declare the message lost.
2. If a consumer reads a message from a topic and then crashes then when the consumer restarts or another instances takes over consumption the new consumer will start from the last known position of the original consumer.

To complete this proposal we just need to figure out how to provide unique PIDs to producers, how to provide fault tolerance for the highwater marks, and how to provide the "fencing" described above to prevent two producers with the same PID from interfering with one another.