
- Look aside = Cache aside
데이터 찾을 때 cache 먼저 확인

정합성 어떻게 할지?
cache down 을 가정하고 성능테스트 필요해보임
cache warming 하는 경우들 있음. 
warming 어떻게 해줄거임?

- Read Through
cache에서만 데이터 읽어옴
데이터 동기화를 동기화 라이브러리, 캐시 제공자에게 위임
(실시간 데이터 조회는 어려워보이네)

조회를 cache에서만 하기때문에 cache failed 되면 서비스에 차질
데이터 동기화, 정합성 차원에서 좀 더 look aside 방식보다 강점이 있나봄

- Write Back, Write Behind - Write Cache Strategy

write를 cache에 해놨다가 db에 반영

db에 write 부하를 줄임
db 에 반영 전에는 유실이 발생할 수 있음

- Write Through
cache -> db에 바로 반영
db 동기화 작업을 cache에 위임
db, cache가 항상 동기화 되어, 데이터 정합성에 강점을 가질 수 있어보임

- Write Around
모든 write를 db에만.

---
- Look Aside + Write Around
일반적으로 많이 씀

- Read Through + Write Around
뭐지 이건. cache에서만 일고,  db에만 쓰는데 어떤 케이스에 쓰나


- Read Through + Write Through
cache에 먼저 write 하여 항상 최신 cache data 보장, cache write -> db write 하여 정합성 보장

---
https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EC%BA%90%EC%8B%9CCache-%EC%84%A4%EA%B3%84-%EC%A0%84%EB%9E%B5-%EC%A7%80%EC%B9%A8-%EC%B4%9D%EC%A0%95%EB%A6%AC