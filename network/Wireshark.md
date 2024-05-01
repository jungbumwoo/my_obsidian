
- 원하는 Column 추가 가능

- Colorize Conversation 으로 보면 원하는 것만 보기 편함

![[Pasted image 20240323070321.png]]

- TCP Steam을 선택하면 해당 패킷을 조립해서 보여주고 해당 부분들만 보여준다.(많이 씀) ex. tcp.stream eq 7 와 같이 필터가 씌여지는 듯
- Statistics 란을 얼마나 잘 쓰냐가 중요

필터
- `A!=B` 가 아닌 `!(A==B)`  로 사용함. 
- 문법 잘 몰라도 각 패킷에 마우스 오른쪽 클릭 -> Apply as Filter로 기본적인건 사용가능.
- CIDR로 네트워크 대역을 지정하는 것이 가능함 - ex) 208.98.137.0/24  를 적용하면 208.98.137.xxx 에 적용되는 것들을 볼 수 있음
- http만 보고싶으면 `http` 치니까 바로 나온다.
- `http.host contains com`,`http.host contains naver`  `contains` 도 포함된거 볼 수 있게 keyword 알아두면 좋음.

Edit - Find Packet 도 패킷 찾는데 유용함. Display Filter, Hex, String, Regular Expression 이용해서 찾을 수 있음.


----
Wireshark으로 TCP 장애 분석하기

장애 분석은 아래에서 위로. 로우레벨에서 위로.
NIC 있는 하드웨어단에서 문제가 있을 수 있는지?
여기서 Packet Lost나 Out of Order가 있는지?

Out of Order 발생하면, Retransmit, Dup Ack가 딸려옴.

하드웨어 레벨에서는 Packet Lost, Out of Order
OS 레벨은 Retransmit,  Dup Ack
Application level에서는 Window size
 https://www.youtube.com/watch?v=6sVRW_nN3-U&list=PLXvgR_grOs1AM8xNKin4NPEb-VL7bkSal&index=10


Statistics - I/O graph 확인하기
![[Pasted image 20240501163335.png]]

default는 TCP `tcp.analysis.flags` 를 필터로 TCP Errors로 전부 찍고 있지만 로우레벨 에러부터 Display Filter를 추가하면서 보면 됨

![[Pasted image 20240501163804.png]]

`tcp.analysis.ack_lost_segment || tcp.analysis.out_of_order`
로 로우레벨 필터를 추가해본 상황.

이렇게 로우레벨 에러가 확인되면 상위 레벨은 멀쩡할 수 없다.
![[Pasted image 20240501164346.png]]
상위 레이어 에러도 확인됨

![[Screenshot 2024-05-01 at 4.53.31 PM.png]]
안좋은건 다 모여 있는 케이스. zero window가 발생하면서 통신이 안된 상황도 있음


아래 interval을 1sec이 아닌 더 짧게 잡고 분석도 가능

----
I/O 그래프 말고 Session 그래프로 파악도 가능하다

![[Pasted image 20240501165935.png]]
문제가 되는거에 follow tcp stream.

![[Pasted image 20240501170049.png]]
Statistics > TCP Stream Graphs > Stevens  찍어봄

세로축 sequence 번호 증가하는건 전송하는 데이터가 늘어나는걸 의미. 근데 빈 구간이 있음. 송수신을 안했다는 의미. 위 zero window 랑 동일해보인다.

다음 그래프도 구불구불해서 문제가 있는걸 보여줌. 이상적인건 위로 일자로 쭉 뻗은것. 기울기가 낮은건 그만큼 느리다는 의미