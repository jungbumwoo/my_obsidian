

해시 함수란 임의 크기 데이터를 고정 크기 값으로 매핑하는데 사용할 수 있는 함수이다.


충돌 시 open addressing, seperate chaining

chaining 방식은 malloc으로 메모리 오버해드 발생
open addressing 방식은 일종 로드 펙터 이상 넘어가면 급격하게 성능이 저하됨
그래서 open addressing 방식을 택하면 로드 팩터를 작게 잡음

루비, 파이썬은 open addressing
자바, 고는 개별체이닝 방식