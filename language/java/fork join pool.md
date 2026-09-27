
parallel program 

solve problems by "divide & conquer"

if (small enough) sovle problem
else
	split problem. fork new sub-tasks, join all sub-tasks

Applying sub-tasks in parallel 

join: merge result
join occurs in a single thread at each level. not need locks for synchronize 

clients insert new fork-join tasks onto a fork-join pool's shared queued, which feeds "work-stealing" queues managed by worker threads

---

vs ThreadPoolExecutor 

threadPoolExecutor: has many control knobs "knobs"
corePool size , maxPool size , workQueue, keepAliveTime, thread Factory, rejectedExecutionHandler



