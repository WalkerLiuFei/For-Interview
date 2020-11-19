# Docker 资源隔离实现反射方式

## Namespace 

Namespace 可以隔离环境变量，并且，让运行在不同Namespace中的进程相互之间不可见 ，但在同一个Namespace 下的进程相互可见。Namespace 实现的6像资源隔离宝库哦 ，UTS

namespace提供了以下6项隔离: 文件系统，进程编号，主机域名，信号量和共享内存，用户组和网络端口 6个隔离。但是没有完全隔离 比如/proc 、/sys 、/dev/sd*等目录

| namespace | 系统调用参数  |                   隔离内容 |
| :-------- | :-----------: | -------------------------: |
| UTS       | CLONE_NEWUTS  |                 主机和域名 |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| PID       | CLONE_NEWPID  |                   进程编号 |
| Network   | CLONE_NEWNET  |   网络设备、网络栈、端口等 |
| Mount     |  CLONE_NEWNS  |           挂载点(文件系统) |
| User      | CLONE_NEWUSER |               用户和用户组 |

但是Namespace 限制资源变量的问题在于，



## CGroups （Control Group of processes）

利用CGroups 可以限制 Namespace这个环境的资源使用情况。但是利用CGroups只能限制环境内的资源只用情况，但是不能限制同一个宿主机下其他Group使用资源的情况，也就是比如说其他CGroups使用了大量内存，也会导致进程OOM。



### 容器隔离性踩过的坑

在使用容器的时候，大家很可能遇到过这几个问题：

1. 在Docker容器中执行top、free等命令，会发现看到的资源使用情况都是宿主机的资源情况 
2. 在容器里修改/etc/sysctl.conf，会收到提示”sysctl: error setting key ‘net.ipv4….’: Read-only file system”；
3. 程序运行在容器里面，调用API获取系统内存、CPU，取到的是宿主机的资源大小；
4. 对于多进程程序，一般都可以将worker数量设置成auto，自适应系统CPU核数，但在容器里面这么设置，取到的CPU核数是不正确的，例如Nginx，其他应用取到的可能也不正确，需要进行测试。