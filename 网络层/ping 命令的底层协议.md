# ping 命令的底层协议

## ICMP 

ping的底层协议是`ICMP`(Internet control message protacol).  它和IP协议同属于网络层的协议。不同于IP 协议，它一般不会被用来传输data。





其实建立在 IPv4 链路层packet上方的数据帧的。ICMP错误消息的最大长度为576个字节。 



## ICMP 攻击

ICMP 是DOS 攻击的一种。 

原理 : 数据从路由器拥有缓冲功能（带宽）。 但是如果大量ICMP请求涌入router，这个buffer被沾满以后，后面来的data 将被discard 掉。这就是ICMP攻击的原理。

## 数据结构

![ICMP header - General-en.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/ICMP_header_-_General-en.svg/300px-ICMP_header_-_General-en.svg.png)