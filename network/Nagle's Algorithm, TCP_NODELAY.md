
## Nagle's Algorithm
By default, `TCP` uses Nagle’s algorithm to collect small outgoing packets to send all at once. This can cause higher rates of latency.

"조금씩 하나만 보내지 말고, 한번에 모아서 보내라"

게임과 같이 반응 속도가 중요한 경우, Nagle 알고리즘을 쓰지 않는다.


```
**if** there is new data to send **then**
    **if** the window size ≥ MSS **and** available data is ≥ MSS **then**
        send complete MSS segment now
    **else**
        **if** there is unconfirmed data still in the pipe **then** // ack 가 왔으면 buffer에 있는거 바로 보내고 ack가 안왔으면 buffer로 들어간다는 의미인듯.
            enqueue data in the buffer until an acknowledge is received
        **else**
            send data immediately
        **end if**
    **end if**
**end if**
```


 If a process is causing many small packets to be transmitted, it may be creating undue network congestion.

these small segments are collected from TCP buffer and sent in a single segment when acknowledgment arrives. So, the faster the acknowledgment comes back, the faster the data is sent.

## Delayed ACK

Delayed ACK is the destination retaining the ACK segment for the value of the delayed ACK timer, about 200 - 500 ms. Delayed ACK means TCP doesn't immediately acknowledge every single received TCP segment. Several ACK responses may be combined together into a single response, reducing protocol overhead. Delayed ACK is basically a bet taken by the destination betting 200 - 500 ms, that a new packet will arrive before the delayed ACK timer expires.

받는 쪽도 한번에 ack를 일일히 다 하는게 아니라 200ms~500ms 동안은 기다렸다가 ack를 보내게 됨


## Nagle's Algorithm and Delayed ACK Do Not Play Well Together in a TCP/IP Network

Delayed ACKs can help in certain circumstances, such as when using the character echo option in Telnet. If the ACKs are tiny and don't use much bandwidth then Delayed ACK is not of much help


### Solution

client 쪽 - Enabling TCP_NODELAY
TCP implementations usually provide applications with an interface to disable the Nagle algorithm. This is typically called the `TCP_NODELAY` option.

```c
int one = 1;
setsockopt(descriptor, SOL_TCP, TCP_NODELAY, &one, sizeof(one));
```



응답 쪽 - delayed ACK
The interface for disabling delayed ACK is not consistent among systems.

delayed ACK(지연된 ACK)를 비활성화하는 방법이 다양한 시스템 간에 통일성이 없음. 서로 다른 운영 체제나 네트워크 시스템에서는 delayed ACK를 비활성화하기 위한 설정이나 명령이 서로 다를 수 있어서, 특정 기능을 일관되게 비활성화하려면 각 시스템에 따라 다른 방법을 사용해야 함. linux 에서는 TCP_QUICKACK.

socket 생성 시 옵션으로 nagle  algorithm을 사용하지 않는 것이 보다 편할 수 있음.





### 아직 덜 이해된 부분

- The user-level solution is to avoid write–write–read sequences on sockets. Write–read–write–read is fine. Write–write–write is fine. But write–write–read is a killer. So, if you can, buffer up your little writes to TCP and send them all at once. Using the standard UNIX I/O package and flushing write before each read usually works. (wiki)


위키가 명쾌했음. 돌고 돌아 다시 위키로 왔는데 위키가 정리를 잘 해두었음.

https://en.wikipedia.org/wiki/Nagle%27s_algorithm


https://www.extrahop.com/company/blog/2016/tcp-nodelay-nagle-quickack-best-practices/

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_for_real_time/8/html/optimizing_rhel_8_for_real_time_for_low_latency_operation/assembly_improving-network-latency-using-tcp_nodelay_optimizing-rhel8-for-real-time-for-low-latency-operation

https://iponwire.com/tcp-nagle-algorithm/

https://jvns.ca/blog/2015/11/21/why-you-should-understand-a-little-about-tcp/

https://gocardless.com/blog/in-search-of-performance-how-we-shaved-200ms-off-every-post-request/