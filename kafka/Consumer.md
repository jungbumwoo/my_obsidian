
-  _onPartitionsRevoked_  rebalance 발생해서  partition 뺏길 때 호출. 뺏기기전 커밋할 때 씀.
-  _onPartitionsAssigned_ rebalance 이후 partiton 새로 부여되면 호출. 필요한 경우 갖다 쓰면 됨.

confluent go client 는 지원해주지 않음;;


