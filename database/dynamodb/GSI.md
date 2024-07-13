
- LSIs provide two read consistency options: _eventually consistent_ (default) and _strongly consistent_ reads.
- All reads from GSIs and streams are eventually consistent.
- Strongly consistent reads are only supported on tables and local secondary indexes. Strongly consistent reads from a global secondary index or a DynamoDB stream are not supported.
-  Strongly consistent reads are only supported on tables and local secondary indexes. Strongly consistent reads from a global secondary index or a DynamoDB stream are not supported.

https://aws.amazon.com/ko/blogs/big-data/scaling-writes-on-amazon-dynamodb-tables-with-global-secondary-indexes/