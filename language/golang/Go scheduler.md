
### goals

- use a small number of kernel threads.
- support high concurrency
- leverage parallelism i.e. scale to N cores.
	- On a N-core machine, Go programs should be able to run N goroutines in parallel.*


### 운영 체제는 이미 **스레드를 관리하는 스케줄러**를 제공하는데, 왜 Go는 자체 스케줄러가 필요할까?

#### ** 운영 체제 스케줄러의 문제점**

운영 체제 스케줄러는 **POSIX 스레드(Pthread)** 기반으로 작동하며, 다음과 같은 특징과 문제점이 있다:

1. **불필요한 기능으로 인한 오버헤드**
    - 운영 체제는 스레드마다 많은 기능을 제공함:
        - **시그널 처리(signal mask)**
        - **CPU 할당(CPU affinity)**
        - **리소스 사용 추적** 등
    - 하지만 Go에서 사용하는 **고루틴(goroutines)**은 이러한 기능이 필요하지 않음.
    - 프로그램에 **10만 개 이상의 고루틴**이 있으면, 운영 체제의 스레드 스케줄러가 이런 불필요한 기능들로 인해 성능이 크게 저하됨.
2. **운영 체제가 Go의 동작 방식을 모른다**
    
    - Go는 **고루틴과 가비지 컬렉터(Garbage Collector, GC)**를 사용.
    - 가비지 컬렉터는 메모리를 회수하기 전에 **메모리 상태가 일관된 상태**여야 함.
        - 이를 위해 모든 스레드를 멈춰야 하는데, 운영 체제는 각 스레드가 어떤 작업을 하고 있는지 알지 못함.
        - 따라서 **무작위 시점에서 멈춘 스레드**들이 일관된 메모리 상태에 도달할 때까지 **오랜 시간 대기**해야 할 수 있음.

#### **Go 스케줄러의 장점**

1. **불필요한 리소스 제거**
    - Go 스케줄러는 불필요한 기능(시그널 처리, CPU 할당 등)을 제거함.
    - 이로 인해, **수십만 개의 고루틴을 가볍게 실행**할 수 있음.
2. **가비지 컬렉터와의 호환**
    - Go 스케줄러는 **고루틴이 멈출 안전한 시점**을 알고 있습니다.
        - 메모리가 일관된 상태인 시점에만 고루틴 실행을 중단.
        - 따라서 **필요한 최소한의 스레드만 기다리게** 하여 가비지 컬렉션 속도를 높임
          
        메모리  GC 실행 중 영향을 주지 않는 task를 수행하는 goroutine은 실행될 수 있게하는듯? (이 부분은 추측)
3. **고루틴 특화 스케줄링**
    
    - Go 스케줄러는 **GMP 모델**(Goroutines, OS Threads, Processors)을 사용하여 고루틴을 OS 스레드 위에서 효율적으로 실행.
    - **OS 스레드보다 훨씬 가벼운 고루틴**을 관리하고, 동시성 퍼포먼스가 좋음.


https://povilasv.me/go-scheduler/
https://www.cs.columbia.edu/~aho/cs6998/reports/12-12-11_DeshpandeSponslerWeiss_GO.pdf

https://morsmachine.dk/go-scheduler

Analysis of the Go runtime scheduler
https://www.cs.columbia.edu/~aho/cs6998/reports/12-12-11_DeshpandeSponslerWeiss_GO.pdf