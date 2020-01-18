# Golang Channel原理

## channel的整体结构图



![](https://i6448038.github.io/img/channel/hchan.png)

- `buf`是有缓冲的channel所特有的结构，用来存储缓存数据。是个循环链表
- `sendx`和`recvx`用于记录`buf`这个循环链表中的~发送或者接收的~index
- `lock`是个互斥锁。
- `recvq`和`sendq`分别是接收(<-channel)或者发送(channel <- xxx)的goroutine抽象出来的结构体(sudog)的队列。是个双向链表

源码位于`/runtime/chan.go`中(目前版本：1.11)。结构体为`hchan`。



每一步的操作的细节可以细化为：

- 第一，加锁
- 第二，把数据从goroutine中copy到“队列”中(或者从队列中copy到goroutine中）。
- 第三，释放锁



## 当channel缓存满了之后会发生什么？这其中的原理是怎样的？

goroutine的阻塞操作，实际上是调用`send (ch <- xx)`或者`recv ( <-ch)`的时候主动触发的，具体请看以下内容：

```go
//goroutine1 中，记做G1

ch := make(chan int, 3)

ch <- 1
ch <- 1
ch <- 1
```

这个时候G1正在正常运行,当再次进行send操作(ch<-1)的时候，会主动调用Go的调度器,让G1等待，并从让出M，让其他G去使用