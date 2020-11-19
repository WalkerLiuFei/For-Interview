# FastHttp中的Golang优化技巧

## FastHttp中的技巧

1. 复用Goroutine
2. fasthttp避免绝大部分多余的内存分配行为，能复用绝不分配。
3. 善用sync.Pool。
4. 尽量避免[]byte与string之间转换带来的开销。
5. 巧用[]byte相关的[特性](https://link.zhihu.com/?target=https%3A//github.com/valyala/fasthttp%23tricks-with-byte-buffers)。
6. DNS cache , 过期时间默认一分钟

## 请求，Gorutine的复用

[server.go](https://github.com/valyala/fasthttp/blob/master/server.go)

[workerPool.go](https://github.com/valyala/fasthttp/blob/master/workerpool.go)

`FastHTTP`通过`workerPool`复用`Goroutine` ，在启动`server`时可以指定server 可以接受的最大并发量, 当一个连接进来以后`server` 会将请求处理交给`workerPool`处理，如果

1. 启动`Server`，启动`WorkerPool`设置最大的`MaxWorkersCount` ,默认最大值 为 ：`256 * 1024`
2. worker当HTTP进来以后，server 首先去查询是否有处于`ready`状态的worker
   1. 如果没有，worker查询worker数量是否达到`NaxWorkersCount`数量，如果达到了，则拒绝这个tcp连接，如果没有则新创建一个worker，并立即执行`HTTP`请求，（ServerHandler 函数），这里新建worker还用到了临时对象池·**`sync.Pool`**也就是代码中的wp.workerChanPool,能在两次gc之间复用对象，减少内存分配的开销,可以看出，这种layzloading的方式，也减轻了内存的负载
3. worker在处理完请求后，会通过realease方法将worker释放到`ready`队列中，这样再下一个请求过来后可以重用这个worker
4. 如果release的worker 在Idea time中还是没有被重用，那这个worker就会被释放，节省内存占用。（单独的一个Goroutine 定时检查）

## 复用 []byte

https://zhuanlan.zhihu.com/p/75929664

减少[]byte的分配，尽量去复用它们

两种方式进行复用：

1. **sync.Pool : 可以缓存，复用对象，sync.Pool可以很好的利用GC，如果sync.Pool中的对象在下一次GC中还没有进行**,fasthttp里共有35个地方使用了sync.Pool，包括上面的。sync.Pool除了降低GC的压力，还能复用对象，减少内存分配。
2. slice = slice[:0]。所有的类型的Reset方法，均使用此方式。例如类型URI、Args、ByteBuffer、Cookie、RequestHeader、ResponseHeader等。还有复用已经分配的[]byte。`s = s[:0]`和`s = append(s[:0], b…)`这两种复用方式，



## 尽量使用 []byte 作为传参

方法参数尽量用[]byte. 纯写场景可避免用bytes.Buffer，方法参数使用[]byte，这样做避免了[]byte到string转换时带来的内存分配和拷贝。毕竟本来从net.Conn读出来的数据也是[]byte类型。



## DNS caching

DNS caching过期时间为1分钟，这样可以避免在大并发场景下每次都进行DNS查询

## 在多核CPU中的优化

1. 使用 `Reuseport` : https://godoc.org/github.com/valyala/fasthttp/reuseport#example-Listen
2. 在每个CPU核心上启动一个`server`实例，并将其设置为 ` GOMAXPROCS=1` ,这样可以减少协程的调度
3. 在Linux系统中 通过[taskset](https://linux.die.net/man/1/taskset) 命令设置进程的CPU亲和性。

