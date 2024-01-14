

![[Pasted image 20240114122253.png]]

- Source connectors interface with the source API and extract the payload + schema of the data, and pass this internally as a generic representation of the data.
- Sink connectors work in reverse—they take a generic representation of the data, and the sink connector plugin writes that to the target system using its API.

### Converters

Converters are responsible for the serialization and deserialization of data flowing between Kafka Connect and Kafka itself.


### Transforms
Unlike connectors and converters, these are entirely optional.


### standalone mode vs distributed mode

In standalone mode, Kafka Connect runs on a single _worker_
In distributed mode, connectors and their tasks are balanced across multiple workers.



----

https://developer.confluent.io/courses/kafka-connect/how-connectors-work/

https://docs.confluent.io/platform/current/connect/devguide.html