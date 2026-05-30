
---

Is Redis Really Single-Threaded? A Comprehensive Analysis from Source Code Perspective  

https://dev.to/deepin/is-redis-really-single-threaded-a-comprehensive-analysis-from-source-code-perspective-211  

  

Summary

Thread/Process Responsibility

Main thread Event loop, command execution

bio thread 1 Asynchronous file closing

bio thread 2 Asynchronous AOF fsync

bio thread 3 Asynchronous lazy freeing

Child process RDB persistence, AOF rewrite

---

  

HashSlot = CRC16(key) mod 16384  

  

key 생성했을 때 샤드에 균등 분배가 되는걸 어떻게 보장?

  

샤드 분배 문제 <> 쿠폰 수집가의 문제.

몇개의 키를 발급하면, 몇 % 확률로 모든 샤드에 키를 균등하게 분배할 수 있는가?

  

Troubleshooting Redis

https://support.redislabs.com/hc/en-us/sections/26758971861778-Troubleshooting-Redis-Software  

  

redis 스레드 몇개? 최적화하려면 8코어 cpu쓰는게 맞나?

  

  

how lazy freeing works?

  

https://medium.com/@adarshmishra98277/the-speed-of-redis-understanding-its-single-threaded-model-2dd5292ebd1b  

  

https://medium.com/@harishsingh8529/how-redis-beats-multithreaded-databases-with-just-one-thread-b60c2d908a58  

  

https://medium.com/@adarshmishra98277/the-speed-of-redis-understanding-its-single-threaded-model-2dd5292ebd1b  

  

  

----

  

모든 키를 hot key로 방지하고자 샤딩처리하면 메모리 오버헤드가 좀 클 것 같은데, hot key 어떻게 동적으로 감지하고 등록할 수 있을까?

  

주기적으로 hot keys 조회 batch가 돌면서 hot 키를 감지하고, (hot key - command는 그럼 어떻게 구현되어 있나? 핫키 return시 어떤 데이터를 return하나? key만 return 하나?)

  

키 설계를 client 에서 호출로 직접 생성이 아닌 레이어를 두고 생성 시 뭔가 validation을 두는게 좋을듯 .

  

application에서 redis cluster를 먼저 샤딩하는건 어떨?

  

kafka도 너무 데이터 input이 많으면 클러스터를 늘리듯이 redis cluster를 병렬로 늘리는거임. 

이럴 때 어떤 메모리 사이즈 기준으로, 키 사이즈에 따라 키는 몇개정도?

---

  

16384 slot 을 쓰는 이유?

  

heart beat packet 보낼 때 소유하고 있는 슬롯 bitmap을 주고 받음. 

16384 % 8 = 2048.

2KB 면 주고 받음. 이정도면 네트워크 사용량이 크지 않은 편.

max cluster node를 임계치인 1,000개로 늘렸을 때, 각 cluster당 약 16개 slot으로 적절하게 분산될 수 있음.

  

---

무중단으로 redis single에서 cluster로 어떻게 migration?

---

여러 개의 키를 하나의 트랜잭션(MULTI)으로 묶거나 조인해야 한다면, 그 키들이 반드시 같은 슬롯(노드)에 있어야 하므로 클러스터 환경에서는 키 설계가 성능과 직결되는 가장 중요한 요소

---

redis에서 cluster 구성을 어떻게 하냐에 따라, 어떤 slot을 어떻게 배치할지도 달라질듯?

cluster가 rack-zone awareness 로 구성되면 slot 배치는 어캐하누?

  

---

  

타임라인의 키값들은 보통 timeline:user:12345:home, timeline:user:12345:mentions 처럼 앞부분이 겹치는 경우가 많습니다.

  

Memcached는 이를 각각 독립적인 문자열 키로 저장하여 중복된 접두사만큼 메모리를 낭비하지만, Redis는 내부적으로 ZipList나 Hashes 등을 통해 이런 공통된 부분을 압축하거나 효율적으로 관리하는 메커니즘을 가지고 있어 메모리 점유율을 낮출 수 있습니다

  

 jb: 오?

  

---

  

ZRANGEBYLEX : Redis Sorted Set에서 점수(Score) 대신 멤버(Member)의 사전식 순서(Lexicographical order)를 기준으로 범위를 지정하여 요소를 조회하는 명령어. ZRANGE에 통합되었나?  

  

이거 생기기 전에는 btree 만들어서 썼다고 함.

  

---

  

redis bandwidth 어떻게 잡을래?

  

usecase를 변경하거나 추가하는거면 current 사용량 모니터링하다가 아래 네개 고려하여 잡으면 되고, 처음 띄우는거면 아래 네 가지를 고려해야하지 않나 싶군.

  

- Client command traffic (requests and responses)  
    
- Replication traffic (primary to replicas)  
    
- Cluster gossip traffic  
    
- Snapshot transfer during replica sync  
    
- monitoring bandwidth (to prometheus)

  

  

Alert when bandwidth exceeds 80% of NIC capacity.

  

  

https://oneuptime.com/blog/post/2026-03-31-redis-how-to-plan-redis-network-bandwidth-requirements/view#:~:text=Plan%20Redis%20network%20bandwidth%20by%20calculating%20per%2Dcommand,detect%20bandwidth%20saturation%20before%20it%20impacts%20latency.  

  

  

---

  

expiration policy 

redis에 lru cache 적용해서 redis에 맡기는 케이스도 있는듯?

  

volatile-lru: Removes least recently used keys that have an expiration set.

allkeys-lru: Removes least recently used keys (even if they don’t have an expiration).

volatile-ttl: Removes the keys with the shortest remaining TTL first.

  

  

---

  

db 를 쓰면 batch 처리를 고민하거나 n+1문제를 고려해야하나 레디스는 이런 문제는 고민하지 않아도 되는 장점이 있음

  

---

  

rate limit 을 redis로 구성 시, rate limit max치는거, rate limit 부하도 생각을 해야함. 

rate limit은 얼마나 많은 client 요청을 받을 수 있나?

  

---

  

Redis persistent RDB(snapshot) / AOF (Append only file)

  

어떤 방식이든 db flush 되기 전 데이터는 유실가능성이 있다고 봐야할듯

AOF도 명령어 실행전에 메모리 상에 로그를 남기지만, 해당 로그를 flush해서 디스크에 보관하기 전에는 유실이 남

  

AOF는 log file 사이즈 기준으로 rewrite를 한다는걸로 보아, 일정 시점 데이터 지난건 날리고 최신을 보존한다는 기조인듯.

특정 snap shot + log가 아니라 old log는 알아서 날리고 최신 log 기반으로 복구한다.

  

유실률은 AOF가 낮을듯? 장애 상황 직전까지 모든 데이터가 보장되어야 할 경우 AOF 사용

출처: https://inpa.tistory.com/entry/REDIS-📚-데이터-영구-저장하는-방법-데이터의-영속성 [Inpa Dev 👨‍💻:티스토리]

  

백업은 필요한데 어느정도 손실이 발생해도 된다? RDB만 사용

RDB가 복구는 더 빠름

  

AOF의 경우 직접 파일 변경이 가능함. 파일 병경 후 복구가 가능하기에 복구 시 제어가 좀 더 용이한 편.

  

AOF도 fork를 사용한다네;

  

If you need to persist the data, run a slave and use that to persist data as it will cause less of a slowdown.  

  

---

  

Pipelining - send multiple commands for network round time 

ㄴ atomic은 보장하지 않음.

atomic 필요하면 Lua Script 짜야함

  

---

  

sentinel - automatic failover, HA

  

---

  

공식 가이드 문서

1. [Persistence and Durability](https://redis.io/tutorials/operate/redis-at-scale/persistence-and-durability/) - RDB and AOF
2. High Availability ← You are here

  

3. [Scalability](https://redis.io/tutorials/operate/redis-at-scale/scalability/) - Redis Cluster
4. [Observability](https://redis.io/tutorials/operate/redis-at-scale/observability/) - Metrics and troubleshooting
5. [Course Conclusion](https://redis.io/tutorials/operate/redis-at-scale/course-wrap-up/)

  

---

유실되면 안되는 경우 WAIT을 쓰면 될듯. - >  ㄴㄴ 이거 쓴다고 strong consistency를 보장하는 건 아닌듯. (https://redis.io/docs/latest/commands/wait/)

  

WAIT

This command blocks the current client until all the previous write commands are successfully transferred and acknowledged by at least the number of replicas  

  

---

  

redis replica에서 read도 가능하지만 cluster 구성하는게 더 쉬울 수도 있다고 공식에서 얘기함

  

---

  

active - active로 locally 하게 write하면 global하게 전파된다고 함. 이거 뭐 ddb 랑 비슷하네. 이건 어캐하는거누

  

  

---

  

이건 나중에 monitoring 필요할 때

https://scalegrid.io/blog/redis-monitoring-metrics/  

  

---

  

how do you _know_ if your key is on the same slot (and the same node/shard) as another key in a transaction?  

https://redis.io/blog/redis-clustering-best-practices-with-keys/

  

지금 내가 할당하려는 키가 어떤 슬롯에 들어갈지 알면, 슬롯 내 키가 편차가 있는지는 서버 런타임에 알 수 있나? 적절한 슬롯에만 키가 몰리지 않도록. 핫 슬롯 방지는 어캐하지?

  

slot 다른데 cluster 같다고 한 트랜잭션으로 묶을 수 있다는 생각은 위험한듯? 언제 slot 이 cluster 이동되지 않는다는걸 보장하기 어려우니까.

  

---

  

redis는 hot keys를 어떻게 감지하고 관리할까?

다른 시스템에 적용해서 모니터링해줄 수 있지 않을까싶음

  

---

  

hot key resolve

  

#### **A. 데이터 크기 불균형 (Big Keys)**

특정 키 하나가 수백 MB를 차지하거나, 특정 슬롯에 수만 개의 키가 몰린 경우입니다.

- **분석 도구:** `redis-cli --bigkeys` 명령어를 통해 메모리를 가장 많이 잡아먹는 상위 키들을 찾습니다.
    

#### **B. 접근 빈도 불균형 (Hot Keys)**

데이터 크기는 작아도 특정 키에 요청이 초당 수만 건씩 몰리는 경우입니다.

- **분석 도구:** `redis-cli --hotkeys` (단, `LFU` 맥스메모리 정책이 설정되어 있어야 함) 혹은 `MONITOR` 명령어로 실시간 유입 쿼리를 분석합니다.
    

---

  

hot key Key Salting  

#### **① Hashtag 사용 최소화 및 재설계**

가장 흔한 원인입니다. 트랜잭션을 위해 너무 많은 데이터를 한 해시태그(`{user123}`)로 묶었다면, 이를 쪼개야 합니다.

- **해결:** 정말 원자적 연산이 필요한 데이터만 태그로 묶고, 나머지는 일반 키로 분산시킵니다.
    

#### **② Big Key 분시 (Splitting)**

하나의 거대한 `Hash`나 `List`가 문제라면 이를 여러 개의 키로 쪼갭니다.

- **예시:** `user:123:followers` (100만 명) -> `user:123:followers:1`, `user:123:followers:2` ... 등으로 분리 저장.
    

#### **③ Key Salting (Hot Key 대응)**

앞서 언급했듯이, 읽기 요청이 몰리는 Hot Key 뒤에 무작위 접미사를 붙여 여러 노드에 복제본을 만듭니다.

- **방법:** `global_config` -> `global_config:1` ~ `global_config:10`으로 복제하고 클라이언트는 랜덤하게 읽음.
    

#### **④ 수동 슬롯 재배치 (Manual Resharding)**

특정 노드가 물리적으로 사양이 낮거나, 담당 슬롯에 유독 무거운 데이터가 많다면 슬롯을 수동으로 옮깁니다.

- **방법:** `redis-cli --cluster reshard` 명령을 통해 부하가 높은 노드의 슬롯 일부를 한가한 노드로 이전합니다.
    

#### **⑤ Read Replicas & Client-side Cache**

- **Read Replicas:** 읽기 요청이 문제라면 마스터가 아닌 복제본(Replica) 노드에서 데이터를 읽어가게 설정합니다.
    
- **L1 Cache:** 어플리케이션 메모리에 핫 데이터를 아주 짧은 시간(TTL 1~5초)만 캐싱해도 Redis로 오는 부하를 극적으로 줄일 수 있습니다.
    

  

---

  

hot key salt를 뒤에 붙이는게 읽을 때 쉬울듯? `user-domain-*` 로 조회하면 알아서 redis가 조회해주지 않을까?

redis는 자료구조 설계상 이렇게 읽을 때 잘 처리해주도록 되어있다며? 어떤 자료구조인데?

  

---

  

redis streams & redis key 분산 실무 & 쿠폰 수집가의 문제

  

라인블로그

https://cwiki.apache.org/confluence/display/KAFKA/KIP-447%3A+Producer+scalability+for+exactly+once+semantics  

  

---

  

skip list 관련 코드 주석

 * This skiplist implementation is almost a C translation of the original

 * algorithm described by William Pugh in "Skip Lists: A Probabilistic

 * Alternative to Balanced Trees", modified in three ways:

 * a) this implementation allows for repeated scores.

 * b) the comparison is not just by key (our 'score') but by satellite data.

 * c) there is a back pointer, so it's a doubly linked list with the back

 * pointers being only at "level 1". This allows to traverse the list

 * from tail to head, useful for ZREVRANGE. */

  

---

  

CRC16 will return a 14-bit number we can then modulo by 16384.  

https://redis.io/blog/redis-clustering-best-practices-with-keys/  

  

---

redis transaction을 쓰기 위해선 동일 slot에 배치가 되어야함.

hashtag를 쓰면 동일 slot에 저장되도록 보장해줌. 특정 노드가 hot spot이 되지 않도록 유의가 필요해보임.

핫키 방지로 키 분산처리해놓은 경우는 쓰기 힘들듯?

  

use case 가 좀 다른 것 같다. 핫키 방지로 키 분산을 해놓고 다시 트랜잭션 처리가 필요한 경우는 아키텍처 설계나 키 설계를 처음부터 다시 논의해봐야하는게 아닐까?

  

---

Q. how redis pub/sub works?

---

  

Q. redis 왜 데이터 유실됨?

A. In-memory 기반이니까, snapshot 방식이 replication이라면 snapshot 생기기전, AOF라면 fsync 되기 전 데이터는 유실될 수 있지 않을까요?

replica 복제가 async 비동기 방식이다보니 유실이 발생할 수 있다.

  

fsync every write (appendfsync always) 설정도 있긴하네. AOF fsync 자체가 비동기이기때문에 이거로도 모든 데이터가 유실되지 않는다고 보기 어려워보인다.  

  

  

  

---

  

Creating a .rdb file requires a lot of disk I/O. If performed in the main Redis process, this would reduce the server's performance. That's why this work is done by a forked child process. But even forking can be time-consuming if the dataset is large.


---

redis-cli --hotkeys 로 hotkey 조회하는데 내부적으로 어떻게 관리?

