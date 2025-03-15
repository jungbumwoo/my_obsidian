

### Redo log

Redo log application is performed during initialization, before accepting any connections. If all changes are flushed from the [buffer pool](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_buffer_pool "buffer pool") to the [tablespaces](https://dev.mysql.com/doc/refman/8.4/en/glossary.html#glos_tablespace "tablespace") (`ibdata*` and `*.ibd` files) at the time of the shutdown or crash, redo log application is skipped. `InnoDB` also skips redo log application if redo log files are missing at startup.



### binlog

The binary log contains “events” that describe database changes such as table creation operations or changes to table data. It also contains events for statements that potentially could have made changes.

- For replication, the binary log on a replication source server provides a record of the data changes to be sent to replicas. The source sends the information contained in its binary log to its replicas, which reproduce those transactions to make the same data changes that were made on the source. See [Section 19.2, “Replication Implementation”](https://dev.mysql.com/doc/refman/8.4/en/replication-implementation.html "19.2 Replication Implementation").
    
- Certain data recovery operations require use of the binary log. After a backup has been restored, the events in the binary log that were recorded after the backup was made are re-executed. These events bring databases up to date from the point of the backup. See [Section 9.5, “Point-in-Time (Incremental) Recovery”](https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery.html "9.5 Point-in-Time (Incremental) Recovery").

Binary logging is done immediately after a statement or transaction completes but before any locks are released or any commit is done. This ensures that the log is logged in commit order.