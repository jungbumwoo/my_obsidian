
pod에 istio, django container 실행되고 있던 상황
istio resource가 django에 비해 크게 리소스 할당이 되고 있었음
django container oom이 발생하는데 pod 단위로 모니터링 하면 oom이 날만큼 문제가 발생하는걸 인지할 수 없었음. 