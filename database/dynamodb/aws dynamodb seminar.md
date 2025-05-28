
오라클 썻었음. multi hour 장애 주문 발생하면서 발생.  
database 스케일 아웃 작업하다 장애 발생포스트모템했더니쿼리 프로파일링 해보니  
70% 싱글 테이블, 싱글 item 조회  
20% 몇10%만이 멀티테이블 조회실제 SA 로 보면 07년 paper publish 이랑 ddb는 다르다.카산드라 하이레벨 보면 똑같음. 사용법 똑같음.  
빅테이블 논문 읽어보면 좋을듯10gb 싱글디비랑 비슷aws, s3, 키네시스 스트림  
tier 0 service. 기반 기술.ddb 쓰는 이유.  
scale 때문에 dynamodb.  
devops 문화 때문.  
aws 2 pizza team.  
aws 안에 인프라 조직이 따로없다. 개발하고 인프라까지 운영 직접해야함. 일반적으로 ddb  
ddb 안쓰려면 굉장히 윗단 까지 가서 허가 받아야함.  
ddb 쓰니 ddb 멈추면 연쇄적으로 무너지는 구조.  
키네시스 스트림 다운된 적 있음. aws 몇십개 서비스 다운됨. ddb 는 파급력이 더 클 것.SDA 분들 돈 많이 받지만  
굉장히 피폐해 보입니다. 뭔가 하나 할려면 문서를 많이 써야한다.  
코드 개발도 해야하지만 만든거 운영, 온콜,내가만든 코드는 위에서 배포일 정해지면 배포담당자 따로 지정  
롤백작업도 해놔야 배포 가능쓰는 이유. 스케일. 스케일 필요 없으면 안써도 된다. 스케일 중요하다.slack을 제외하고 메세징, 채팅 대부분 회사는 key-value 기반

1. 서비스 쿼타는 높여놔야함. 2. 프리 파티셔닝 되어있으면 좋음

--------------------------------------디지니플러스. global table 사용함.  
추천리스트 담는 용도. 꼭 보셨으면 좋겠다.click, acitviy 키네시스로 수집.  
ml 로 개인 리스트 뽑아서 특정 리전에서 aws 로 write  
global 하겍 복제됨,---------------------------------ddb 도 cahce layer 가 있다.  
DAX adds read cache.  
쓰면서 만족하는 고객을 만나본적 없다.  
나중에 나오더라도 많이 고민해보셨으면 한다.앞에 es 캐시 쓰는걸 권장.DAX 인프라 관리. replica 붙였다 떌 수 있다.  
스케일 업 할 때 in-place update 안됨  
fail over 발생하는데 그 동안 write 가 안됨.  
올해 리인벤트에 DAX 대체할거 나올거다.------------------mission-critial oltp.  
풀 매니지 서비스.rdms 장점정규화 왜 함?  
테이터 중복 줄이기 위해서.  
지금은 디스크 싼데 왜 줄여야하죠?natural join 동작방식. 시대적 배경.  
조금씩 들고와서 loop 돌면서 temp table 만들고 return정규화 장점.  
규칙이 정해져있음. 그래서 이해하기 쉬움.nosql. 규칙, 공식이 없어서 어려워함. 패턴은 있어서 오늘 배우고why nosql?  

- large data processing.
- 2 pb 저장해야한다. 어디 저장할래?
- 시계열 데이터. 쓰기만 잔뜩하고 안읽음.
- 모니터링 서비스들 그래서 다 key-value

카산드라 탑 고객 2  
애플, 넷플릭스.애플은 왜 key-value를 많이 쓸까?  
30만대 카산드라 씀. 아이클라우드, imessage넷플릭스 개발자 블로그보면 카산드라를 자동화해서 ddb 처럼 씀-------  
일정한 레이턴시 보장. 광고회사에서 많음  
일기보다 쓰기가 많은 케이스Q.  
시간이 없어서 설명하지 못한 부분, 읽어보면 좋d은 아티클Q.  
채팅 모델링.Q. consistent hash ringQ. 이런 세션 또 참여하려면?슈퍼볼 미리 스케일링-----------------  
32x 마지막까지 db 썼다. 더 스케일 필요하면 어떻게 할까?  
시간 벌어야되니까  
보통 쿼리 튜닝. 그 다음 샤딩.샤딩 어려운점:  
다시 샤드 늘릴 때.  
meta data 관리.  
장애났음. 샤드 복구.  
샤딩 되면 여러 사람이 힘들어짐. dba가 힘들어짐. ddl 여섯번 때려야하고,..slack 기술 블로그 보면 샤드 많이 쓰는데, 이런 작업할 수 있는 인력 소싱 가능근데 한국은?  
한국엔지니어, 서울경기. 시니어 dba, 대규모트래픽 경험 - 조건 부합하는 사람 찾기 쉽지않음.  
dba 특히 요새는 많이 안하는 추세같아서 더 찾기 힘듬.그럼 샤딩 결정할 수 있나? 하고 퇴사하면 가능. 그렇지 않으면 백업, 복구 힘들거임. 그래서 mangge servivce.nosql.  
redis 정규화 안하지 않냐.  
얼마나 빨리 서빙할 수 있을지를 고민.ddb 는 샤딩 자동화 아닌가요?Q. partition limit WCU 1k, 10GB . 디자인할 때. 왜 1k. 인지. 시간 지나면 올라갈 수 있는건지.Q. 10억개가량 쌓였다. 진짜 어디까지 저장 가능한지. 왜 그 지점에서 한계가 발생하는지?pk+ sk 같이 쓰는건 redis sorted set. pk 만 쓰면 string key.Scan - table에서 table 마이그레이션 등에서 쓰지  
parall 옵션으로 스캔이 가능.batch - rrt 요청 수 감소를 위해서. unprocess key return 함. 이건 비용안나옴.  
update는 batch 없는 이유: idempotent 지원이 안되기 때문에.transacitonWrite, transactionRead - 비용 두배GSI ALL default 이긴한데 best 가 아닐 수 있음. io 도 줄일 수 있음.  
GSI 는 또다른 테이블이 생성되는 것. 최대 20까지 생성하는건데 도달하면 디자인 문제가 있는 것.GSI는 돈이 든다. 근데 rdb도 index 생성 시 비용발생합니다~LSI 도 공짜는 아니다. 비용에 따로 안잡히는 것 뿐. base table에 같이 써짐.  
추천은 안하는게 쿼리 유연성 떨어짐. 만들때만 생성가능하기 때문. 장점은 최신 데이터 읽을 수 있다.key 난수로 직접만들 필요 없다. dㅏㅇㄹ아서 해싱해줌.AZ 두개 무너지면 쓰기는 안됨. 하나 무저져도 쓰기 읽기 됨.request router 에서 consistent hash ring, 어디로 라우팅할지 meta data 관리read 시 랜덤 라우팅하다가 최근 같은 AZ로 라우팅된다. strong concicset read는 리더에서.target util default 70. spike 때문에 default 90으로는 안해놨음.  
토큰으로 RCU,WCU 남는 토큰, bustbucket에서. 없으면 스로틀링.스로틀링. = rate limit  
ondemand 써도 스로틀링 걸린다. 줄어드는 것 뿐.  
ondemand : do not scale down. 늘어나거나 유지. 인스턴스 비용은 없음. aws에 안좋은 것. 유저한테 좋은 부분.스토리지 테이블  
rcu,wcu 비용, 스토리지비용 차이. 무중단 없이 바꿀 수 있음. 퍼포먼스 차이 없음. 위험부담 없음.  
하루정도 지나서 비용을 보세요.파티션이 아닌 테이블 단위에서 SLA 99.999%ddb 페이지네이션 힘듬. admin에서는 적합하지 않음.ddb는 TTL 는 어떻게 지우는지?



-----  

aws reinvent tinder

ec mongodb -> dynamo. 이 영상은 꼭 보셨으면.

  

이기종간 db  migration

  

  

읽고 싶은대로 저장.

  

쓰기가 많은지.

일정한 latency 보장.

  

  

nosql 은 테이블 구조가 중요하는게 아니라

어떤 파라미터로 읽고 쓸지가 중요.

  

일반적으로는 multi table이 좋다.

  

nosql은 OLAP는 잘 하진 않음.

  

-----------------

  

파티션키는 hashing 하기 때문에 이퀄조건 밖에 못쓴다.

  

RDS와 달리 패턴만 있고 공식이 없다.

  

GSI  pk 값이 원본 테이블에 있는 경우에만 복재 된다.

  

sdk level에서 retry가 다 끝나고 스로틀링 모니터에 잡히는 것.

  

  

  

  

dynamodb streams 

  

내부 파티션 갯수 알 필요 없다.

껏다 켰다 할 수 있음. 켜는 것 비용 없음.

item 가져가면 가져가면 돈나옴.  lambda 로 쓰면 steams 비용은 안나옴

steams에서 old, new, old and new 뭐 받을지 선택 가능

  

dynamodb + lambda 많이 씀

lambda 처리 가능한 캡이 있어서 이거 넘어가면 못함.

  

Q. 엄격하게 TTL 보장이 필요하면 코드에서 한번 더 필터링해줘야하는 상황?

쿼리 시에는 조건에 직접 filter 로직을 작성해줘야함

  

샤드 늘리는거보다 합치는게 어렵다.

  

transaction 은 10분만 보장되는듯?

  

128테라까지 쓸 수 있는데 정규화 안하는 회사들도 있음. join 없이 씀

  

RDS FK 도 스케일이 커지면 안쓰려한다.

  

  

redis: command thread, io thread.

메인스레드는 sequence command 실행만. io thread 가 

  

elastic cache

1.버전을 최신으로 쓴다.

2.클러스터 모드만 쓴다. (싱글은 스케일업 해봤자 command thread  처리가 늘어나지 않는다. 나중에 바꾸려면 갈아엎어야한다.)

  

redis에서 pubsub은 안했으면 좋겠다.

  

redis - > valkey 쓰면 비용 정확하게 20% 감소. 7.2를 권장. 안할 이유가 없다. 빠르게 전환하는게 좋다.

  

pubsub 

  

dynamodb. 버전이 없음. 무중단. 항상 떠있음.

  

user pattern 실제 패턴 예시.

  

당근 기술블로그 feature store "추천의 심장 피쳐스토어"

  

  

session store.

2018 reinvent: dynmamodb underthe hood

  

elastic cahce 샤딩해서 session 쓰더라도 버전업이나 fail over 발생 시 뒷단에 ddb durable 스토리지를 같이 쓴다. 레이어 두개를 둔다.

  

그럼 두개 정합성 어떻게 가져가는지? 하 질문했었어야하는데 골이 띵

  

-----------------------------------------------------------

  

pk, sk string을 쓴다. string은 string, number 둘 다 커버 가능

  

pk, sk 동일 한 값 두개 넣는 경우는 싱글테이블의 경우 pk만 필요한경우, sk 만 필요한 경우 다 있을 수 있으니

  

단독 attr에 전부 넣느냐? 쪼개서 별도 attr에 넣을까? ->  gsi 에서 키로 쓸 수 있는지 

  

싱글테이블 썻을 때 동일 테이블에 조인결과를 같이 넣어드려서

  

  

키 prefix를 둔다.

앞에 messgae 를 둔다. 

  

  

  

  

  

  

  

-----------------

[https://aws.amazon.com/ko/blogs/korea/how-amazon-dynamodb-adaptive-capacity-accommodates-uneven-data-access-patterns-or-why-what-you-know-about-dynamodb-might-be-outdated/](https://aws.amazon.com/ko/blogs/korea/how-amazon-dynamodb-adaptive-capacity-accommodates-uneven-data-access-patterns-or-why-what-you-know-about-dynamodb-might-be-outdated/)