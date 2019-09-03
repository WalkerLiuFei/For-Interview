# Client

如果Redis 响应很慢，很有可能就是有人在操作Redis阻塞了Redis，通过`client`命令查询所有的正在连接的client，查看状态和执行的命令，进行处理



`id=7 addr=10.197.2.101:58739 fd=5 name= age=4 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client`



在下面的场景中 ，`client`会被服务器主动关闭

1. 如果客户端进程退出或者被杀死
2. 客户端向服务端向服务器发送了带有不符合协议格式的命令请求
3. client kill 命令
4. client连接市场超过了 server设置的timeout
5. 客户端发送的命令请求的大小超过了输入缓冲区的限制大小
6. 服务器向客户端回复超过了缓冲区大小

