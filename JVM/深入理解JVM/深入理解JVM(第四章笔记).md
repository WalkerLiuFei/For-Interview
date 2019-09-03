# 深入理解JVM(第四章笔记)



## JDK 性能监控工具：
|名称|主要作用|
|---|-----|
|jps|JVM Process status tool,显示制定系统内所有的Hotspt虚拟机进程
|jstat|JVM Statistic Monitor Tool，收集Hotspot 虚拟机各方面的运行数据|
|jinfo|configuration info for java，显示虚拟机配置信息|
|jmap|memory map for java  生成虚拟机的内存转储快照（heapdump 文件）|
|jhat| JVM Heap dump browser 用于分析heapdump 文件，它会建立一个HTTP/HTML 服务器，让用户可以再浏览器上查看分析结果 |
|jstack|stack trace for java|显示虚拟机线程快照|

### JPS
> jps [ options ] [ hostid ]
> 命令中的hostid 为RMI注册表中注册的主机名

-q    只输出LVMID()，省略执行主类名
-m   输出进程启动时，传给执行主类main()函数的参数
-l    输出主类的全名，如果进行执行的是jar，输出jar路径
-v    输出虚拟机进行启动时的JVM参数
### jstat
> 命令格式： jstat [ option vmid [interval][s | ms] [count]]
>  对于命令中的vmid,如果是本地的虚拟机进程，vmid 和 lvmid是一致，如果是远程的格式为 [protocol:][//]lvmid[@hostname [:port]/servername]


   ```   
    jstat -class pid:显示加载class的数量，及所占空间等信息。  
    jstat -compiler pid:显示VM实时编译的数量等信息。  
    jstat -gc pid:可以显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是young gc的次数，young gc的时间，full gc的次数，full gc的时间，gc的总时间。  
    jstat -gccapacity:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old内纯的占用量。  
    jstat -gcnew pid:new对象的信息。  
    jstat -gcnewcapacity pid:new对象的信息及其占用量。  
    jstat -gcold pid:old对象的信息。  
    jstat -gcoldcapacity pid:old对象的信息及其占用量。  
    jstat -gcpermcapacity pid: perm对象的信息及其占用量。  
    jstat -util pid:统计gc信息统计。  
    jstat -printcompilation pid:当前VM执行的信息。  
   ```
### jmap
> 使用方法 ： jmap [ option ] vmid
+ -dump:[live,]format=b,file=<filename> 使用hprof二进制形式,输出jvm的heap内容到文件=. live子选项是可选的，假如指定live选项,那么只输出活的对象到文件. 
+ -finalizerinfo 打印正等候回收的对象的信息.
+ -heap 打印heap的概要信息，GC使用的[算法](http://lib.csdn.net/base/datastructure)，+ heap的配置及wise heap的使用情况.
+ -histo[:live] 打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”. 如果live子参数加上后,只统计活的对象数量. 
+ -permstat 打印classload和jvm heap长久层的信息. 包含每个classloader的名字,活泼性,地址,父classloader和加载的class数量. 另外,内部String的数量和占用内存数也会打印出来. 
+ -F 强迫.在pid没有相应的时候使用-dump或者-histo参数. 在这个模式下,live子参数无效. 
+ -h | -help 打印辅助信息 
+ -J 传递参数给jmap启动的jvm. 

### 
> jstack  [options] vmid  : 查看线程快照，主要目的是定位线程出现长时间停顿的原因
> -F当’jstack [-l] pid’没有相应的时候强制打印栈信息
> -l长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表.
> -m打印java和native c/c++框架的所有栈信息.
> -h | -help打印帮助信息