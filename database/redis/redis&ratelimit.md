
https://redis.io/blog/api-throttling-algorithms-patterns/


### Fixed Window
경계 지점에 burst 발생할 수 있는 것이 단점

### Sliding Window Log
O(n) 연산 필요. 정확한 카운트 정책 제어. 인증 보안 쪽에서 사용

### Sliding Window Count
Fixed window에서 발생하는 burst를 완화
Key 2개로 처리.  sliding window에 비해서 resourse 사용양도 적음.

### token bucket
일적인 burst 허용. 정책적으로.

원자적 연산을 위해 lua 사용 필요.
token 수. 마지막 사용 시점.

마지막 사용 시점으로 다음 요청에서 토큰이 얼마나 있는지를 계산한다.

### leaky bucket
서버로 일정 트래픽을 흘려보냄. 앞단에 큐를 두고 일정 요청량을 제어한다.