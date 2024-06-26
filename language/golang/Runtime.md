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