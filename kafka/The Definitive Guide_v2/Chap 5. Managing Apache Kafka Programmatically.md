
## AdminClient Overview

### Asynchronous and Eventually Consistent API

- Kafka’s AdminClient is that it is asynchronous.
- Each method returns immediately after delivering a request to the cluster controller, and each method returns one or more Future objects.
- Future objects are the result of asynchronous operations, and they have methods for check‐ ing the status of the asynchronous operation, canceling it, waiting for it to complete, and executing functions after its completion.