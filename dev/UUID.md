
if the node ID changes (e.g., because a network card has
   been moved between machines), setting the clock sequence to a random
   number minimizes the probability of a duplicate due to slight
   differences in the clock settings of the machines.

**같은 시간(밀리초) 내에 많은 UUID를 생성하는 경우**:
- 같은 밀리초 동안 UUID를 생성하려면 타임스탬프가 겹칠 위험
- 카운터를 활용(생성 대기)하거나 추가 노드 ID를 통해 고유성을 유지



https://datatracker.ietf.org/doc/html/rfc4122