# kube-proxy

每台机器上都运行一个 kube-proxy 服务，它监听 API server 中 service 和 endpoint 的变化情况，并通过 iptables 等来为服务配置负载均衡（仅支持 TCP 和 UDP）。

**kube-proxy 可以直接运行在物理机上，也可以以 static pod 或者 daemonset 的方式运行。** 



## kube-proxy 工作原理

kube-proxy 监听 API server 中 service 和 endpoint 的变化情况，并通过 userspace、iptables、ipvs 或 winuserspace 等 proxier 来为服务配置负载均衡（仅支持 TCP 和 UDP）。

![img](https://feisky.gitbooks.io/kubernetes/components/images/kube-proxy.png)

## kube-proxy 不足

kube-proxy 目前仅支持 TCP 和 UDP，不支持 HTTP 路由，并且也没有健康检查机制。这些可以通过自定义 [Ingress Controller](https://feisky.gitbooks.io/kubernetes/plugins/ingress.html) 的方法来解决。

