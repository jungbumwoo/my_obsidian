

Producer restart
When producer starts, if the idempotent producer is enabled, the prodcuer will initialize and reach out a Kafka broker to generate a producer ID. If a prodcuer fails and the producer that replaces it sends a message that was previously sent by the old producer, the broker will not detect the duplicates.

