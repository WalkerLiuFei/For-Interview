# 百万 Go TCP 连接的思考

https://colobu.com/2019/02/23/1m-go-tcp-connection/

## 序言

一般Go语言的TCP(和HTTP)的处理都是每一个连接启动一个goroutine去处理，因为我们被教导goroutine的不像thread, 它是很便宜的，可以在服务器上启动成千上万的goroutine。但是对于一百万的连接，这种**goroutine-per-connection**的模式就至少要启动一百万个goroutine，这对资源的消耗也是极大的。针对不同的操作系统和不同的Go版本，一个goroutine锁使用的最小的栈大小是2KB ~ 8 KB ([go stack](https://github.com/golang/go/blob/release-branch.go1.11/src/runtime/stack.go#L64-L82)),如果在每个goroutine中在分配byte buffer用以从连接中读写数据，几十G的内存轻轻松松就分配出去了。

使用一个goroutine代码一百万的goroutine, 另外使用ws减少buffer的分配，极大的减少了内存的占用，这也是大家热议的一个话题。 