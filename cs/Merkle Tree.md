
- Git
- DynamoDB
- BlockChain


Dynamo uses vector clocks to remove conflicts while serving read requests.
If a replica falls significantly behind others, it might take a very long time to resolve conflicts using just vector clocks.
We need to quickly compare two copies of a range of data residing on different replicas and figure out exactly which parts are different.

