
By default, `TCP` uses Nagle’s algorithm to collect small outgoing packets to send all at once. This can cause higher rates of latency.

"조금씩 하나만 보내지 말고, 한번에 모아서 보내라"

게임과 같이 반응 속도가 중요한 경우, Nagle 알고리즘을 쓰지 않는다.

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_for_real_time/8/html/optimizing_rhel_8_for_real_time_for_low_latency_operation/assembly_improving-network-latency-using-tcp_nodelay_optimizing-rhel8-for-real-time-for-low-latency-operation