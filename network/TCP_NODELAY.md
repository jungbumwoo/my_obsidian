
By default, `TCP` uses Nagle’s algorithm to collect small outgoing packets to send all at once. This can cause higher rates of latency.

"조금씩 하나만 보내지 말고, 한번에 모아서 보내라"

게임과 같이 반응 속도가 중요한 경우, Nagle 알고리즘을 쓰지 않는다.

 If a process is causing many small packets to be transmitted, it may be creating undue network congestion

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_for_real_time/8/html/optimizing_rhel_8_for_real_time_for_low_latency_operation/assembly_improving-network-latency-using-tcp_nodelay_optimizing-rhel8-for-real-time-for-low-latency-operation



Delayed ACK is the destination retaining the ACK segment for the value of the delayed ACK timer, about 200 - 500 ms. Delayed ACK means TCP doesn't immediately acknowledge every single received TCP segment. Several ACK responses may be combined together into a single response, reducing protocol overhead. Delayed ACK is basically a bet taken by the destination betting 200 - 500 ms, that a new packet will arrive before the delayed ACK timer expires.

받는 쪽도 한번에 ack를 일일히 다 하는게 아니라 200ms~500ms 동안은 기다렸다가 ack를 보내게 됨


### Nagle's Algorithm and Delayed ACK Do Not Play Well Together in a TCP/IP Network

