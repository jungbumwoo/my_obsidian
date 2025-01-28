
if the node ID changes (e.g., because a network card has
   been moved between machines), setting the clock sequence to a random
   number minimizes the probability of a duplicate due to slight
   differences in the clock settings of the machines.

**같은 시간(밀리초) 내에 많은 UUID를 생성하는 경우**:
- 같은 밀리초 동안 UUID를 생성하려면 타임스탬프가 겹칠 위험
- 카운터를 활용(생성 대기)하거나 추가 노드 ID를 통해 고유성을 유지

mac address 대체
 임의의 랜덤 값을 노드 ID로 생성

- **47비트 크기의 고품질 암호학적 난수**를 생성해, 이를 노드 ID로 사용할 수 있음
- 생성한 난수의 **첫 번째 바이트의 최하위 비트(Least Significant Bit, LSB)**를 **1로 설정**
    - 이 비트는 **유니캐스트/멀티캐스트 비트**로, MAC 주소와 충돌하지 않도록 설계
    - 네트워크 카드에서 실제로 생성된 MAC 주소와 충돌할 가능성을 없앰


https://datatracker.ietf.org/doc/html/rfc4122