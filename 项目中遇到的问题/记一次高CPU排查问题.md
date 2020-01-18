# 记一次高CPU排查问题

## 问题描述

Golang程序，用于消费kafka集群消息，然后通过简单的计算，将数据缓存到Redis，进而持久化到Mysql中。遇到的问题发现是CPU使用率较高。

## 问题解决过程

### 采集CPU Profile

采集CPU Profile的方式有两种

1. 通过端口形式` https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof `
2. 通过文件形式，我采用的是这一种。。

另外注意通过`runtime.SetCPUProfileRate()` 提高采集效率

### CPU Profile中的指标含义

举个例子：

#### 代码

```go
package main

import (
	"encoding/json"
	"os"
	"runtime"
	"runtime/pprof"
)

func main(){
	cpuprofile := "./cpuprofile"
	f,err := os.Create(cpuprofile)
	if err != nil{
		panic(err)
	}
	runtime.SetBlockProfileRate(1000)
	pprof.StartCPUProfile(f)
	defer pprof.StopCPUProfile()
	sum()
}


func sum(){
	a()
	b()
	c()
}
var data = "1111111111111111"
func a(){
	for count := 0;count < 1000000;count++{
		json.Marshal(data)
	}

}

func b(){
	for count := 0;count < 2000000;count++{
		json.Marshal(data)
	}

}

func c(){
	for count := 0;count < 3000000;count++{
		json.Marshal(data)
	}

}
```



```
     flat  flat%   sum%        cum   cum%
     610ms 51.69% 51.69%      610ms 51.69%  runtime.stdcall1
      80ms  6.78% 58.47%      430ms 36.44%  runtime.timerproc
      40ms  3.39% 61.86%       80ms  6.78%  runtime.exitsyscall
      40ms  3.39% 65.25%      510ms 43.22%  runtime.startm
      30ms  2.54% 67.80%       30ms  2.54%  runtime.casgstatus
      30ms  2.54% 70.34%       40ms  3.39%  runtime.entersyscallblock
      30ms  2.54% 72.88%      150ms 12.71%  runtime.goready
      30ms  2.54% 75.42%      110ms  9.32%  time.Sleep
      20ms  1.69% 77.12%      490ms 41.53%  runtime.goready.func1
      20ms  1.69% 78.81%      190ms 16.10%  runtime.goroutineReady
```

1. flat 和 flat% ： 分别是指对应的函数一共被调用了多少次，被调用的次数占总的样本次数的占比
2. sum% 表示 ：运行总计 :这一个意义不大，例如第三行的 61.86 % 等于第二行的 58.47% + 第三行的flat% ，3.39%
3. cum 和cum %: 表示函数的 调用包括函数里面调用所占的比重,下面即为按照 cum%排序的结果

```
(pprof) top -cum
Showing nodes accounting for 0.30s, 12.66% of 2.37s total
Dropped 33 nodes (cum <= 0.01s)
Showing top 10 nodes out of 92
      flat  flat%   sum%        cum   cum%
         0     0%     0%      1.93s 81.43%  main.main
         0     0%     0%      1.93s 81.43%  main.sum
         0     0%     0%      1.93s 81.43%  runtime.main
     0.08s  3.38%  3.38%      1.57s 66.24%  encoding/json.Marshal
     0.01s  0.42%  3.80%      0.96s 40.51%  main.c
     0.01s  0.42%  4.22%      0.92s 38.82%  encoding/json.(*encodeState).marshal
     0.06s  2.53%  6.75%      0.79s 33.33%  encoding/json.(*encodeState).reflectValue
     0.01s  0.42%  7.17%      0.66s 27.85%  main.b
     0.03s  1.27%  8.44%      0.41s 17.30%  encoding/json.stringEncoder
     0.10s  4.22% 12.66%      0.40s 16.88%  runtime.mallocgc
```



**总之：用flat排序查看那个函数被调用的最频繁，用cum排序查看那个函数耗时最长**

### Go Tool pprof的使用

通过 `(pprof) web mapaccess1` 这样的方式，可以将web中的graph简化，得到一个访问 mapaccess1的图。