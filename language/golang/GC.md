
Trade Throughput for Reduced GC Latency

We're gonna make your program slower, But what we're gonna give you, reduce GC latency

### GC algorithm Phaes

![[Pasted image 20241209012010.png]]

1. OFF
	GC disabled. Pointer writes are just memory writes
2. Stack scan
	처음 시작 시에만 잠깐 STW 가 발생함. Collect pointers from gloabls and goroutine stacks. Statcks scanned at preemtion points
3. Mark
	Mark objects and follow pointers until pointer queue is empty
	Write barrier tracks pointer changes by mutator
	



https://www.youtube.com/watch?v=aiv1JOfMjm0