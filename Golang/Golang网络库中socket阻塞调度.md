# Golang网络库中socket阻塞调度



Go提供的一种强大的抽象`net.Conn`。它不仅仅是代表一个socket，同时它还被封装成了netPoller对象。netPoller是Go runtime的一个数据结构，也许你早已知道了linux的epoll，netPoller就是对epoll的一种封装。

**也就是先会进行一次syscall.Read，但是如果没有数据，此时会得到一个错误syscall.EAGAIN。这时，会执行fd.pd.waitRead，这个函数会一直阻塞直到epoll通知socket有数据就绪。这里的阻塞和syscall的阻塞调用不一样，这里的阻塞相当于主动让出时间片(park)，当前线程可以去执行别的goroutine**





读完这个文章

https://studygolang.com/articles/4977

看一个`conn` 从Socket读取数据的小片段

1. Conn.Read()

2. Conn.Read 其实是conn.Read(), conn是Conn的具体实现，conn的 实际是一个`netFD` 文件描述符`fd`.

   ```
   type netFD struct {
   	pfd poll.FD
   
   	// immutable until Close
   	family      int
   	sotype      int
   	isConnected bool // handshake completed or use of association with peer
   	net         string
   	laddr       Addr
   	raddr       Addr
   }
   ```

3. 最后的读取是 `socket`文件描述符 `poll.FD`完成的。

    ```
   // Read implements io.Reader.
   func (fd *FD) Read(p []byte) (int, error) {
     if err := fd.pd.prepareRead(fd.isFile); err != nil {
   		return 0, err
   	}
   // 没用的代码
   	for {
   	  // 数据ready了，读的结果err就为nil，数据就返回
   		n, err := syscall.Read(fd.Sysfd, p)
   		if err != nil {
   			n = 0
   			// 如果数据没准备好，那就wait.让出时间片。
   			if err == syscall.EAGAIN && fd.pd.pollable() {
   				if err = fd.pd.waitRead(fd.isFile); err == nil {
   					continue
   				}
   			}
   
   			// On MacOS we can see EINTR here if the user
   			// pressed ^Z.  See issue #22838.
   			if runtime.GOOS == "darwin" && err == syscall.EINTR {
   				continue
   			}
   		}
   		err = fd.eofError(n, err)
   		return n, err
   	}
   }
    ```

4. 像上面的`fd.pd.waitRead` 和`fd.pd.prepareRead` ,`syscall.Read(fd.Sysfd,p)`都是和系统相关的操作，所以都是放在`internal`包和`syscall`下面。

5. 在`fd` 没有准备好时候就立即响应一个`EAGAIN` 标识符，这个就是`EPOLL`的标识，所以为什么说golang适合高并发的业务场景。这里既是原因之一，默认在socket W/R，使用epoll提高效率！

6. 如果fd没有准备好read操作，它就执行`fd.pd.waitRead` 这个最后也是在`runtime`包下面执行的:

   其实流程大概就是

   1. 通过CAS操作拿到`pd`

   ```go
   // returns true if IO is ready, or false if timedout or closed
   // waitio - wait only for completed IO, ignore errors
   func netpollblock(pd *pollDesc, mode int32, waitio bool) bool {
   	gpp := &pd.rg
   	if mode == 'w' {
   		gpp = &pd.wg
   	}
   
   	// set the gpp semaphore to WAIT
   	for {
   		//自旋梭，将gpp设置为pdWait状态
   		if atomic.Casuintptr(gpp, 0, pdWait) {
   			break
   		}
   	}
   
   
     // 明显的当 pd没有超时或者waitio为true时，将gorutine挂起到park状态
     // netpollblockcommit是 goroutine的调度用的函数，用途是监控fd状态是 ready
   	if waitio || netpollcheckerr(pd, mode) == 0 {
   		gopark(netpollblockcommit, unsafe.Pointer(gpp), waitReasonIOWait, traceEvGoBlockNet, 5)
   	}
   	// be careful to not lose concurrent READY notification
   	old := atomic.Xchguintptr(gpp, 0)
   	if old > pdWait {
   		throw("runtime: corrupted polldesc")
   	}
   	return old == pdReady
   }
   ```

7. `netpollblockcommit`函数

   ```
   // 同样是个CAS操作，如果是false，说明fd值已经改变，可以进行下面的操作
   func netpollblockcommit(gp *g, gpp unsafe.Pointer) bool {
   	r := atomic.Casuintptr((*uintptr)(gpp), pdWait, uintptr(unsafe.Pointer(gp)))
   	if r {
   		// Bump the count of goroutines waiting for the poller.
   		// The scheduler uses this to decide whether to block
   		// waiting for the poller if there is nothing else to do.
   		atomic.Xadd(&netpollWaiters, 1)
   	}
   	return r
   }
   ```

8. 其他的都是Goroutine的相关操作了。



