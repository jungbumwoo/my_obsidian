
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


 