
- Git
- DynamoDB
- BlockChain


Dynamo uses vector clocks to remove conflicts while serving read requests.
If a replica falls significantly behind others, it might take a very long time to resolve conflicts using just vector clocks.
We need to quickly compare two copies of a range of data residing on different replicas and figure out exactly which parts are different.

![[Pasted image 20240616083212.png]]

Merkle trees is conceptually simple:
1. Compare the root hashes of both trees.
2. If they are equal, stop.
3. Recurse on the left and right children.

https://www.designgurus.io/course-play/grokking-the-advanced-system-design-interview/doc/636cf2447b49c1a061cd7baf

