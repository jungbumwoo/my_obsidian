go binary를 실행하면 main이 처음 실행되는 entry가 아님

entry point is runtime. bootstrap process of runtime.

 run.

- Thread Local Storage
- SYSARGS. 
	- 환경 변수나 os 시작 할 때 args 들이 들어감
- OS - initialize page, signal handling ..
- SCHED
	- big chunk of initialization of the Run time.
		- SCHEDULER
		- 어떤 고루틴이 다음에 실행될지에 대한 책임을 갖고 있음


- The Go runtime manages scheduling, garbage collection, and the runtime environment for goroutines.
- Runtime keeps track of each **G** and maps them onto **Logical Processors**, named [**P**](https://github.com/golang/go/blob/5dd978a283ca445f8b5f255773b3904497365b61/src/runtime/runtime2.go#L470). **P** can be seen as a abstract resource or a context, which needs to be acquired, so that **OS** **thread** ([called **M**](https://github.com/golang/go/blob/5dd978a283ca445f8b5f255773b3904497365b61/src/runtime/runtime2.go#L403), or Machine) can execute **G** .