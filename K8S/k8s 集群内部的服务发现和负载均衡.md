## K8s 服务集群内部的服务发现和负载均衡

## 服务发现

服务发现再K8s 内部是通过`coreDns` 进行的。服务在k8s 中以service的形式提供服务，k8s会为其分配一个`front ip`。而后面的`pod` 的名字是 `EndPoint` IP.

```yaml
Name:                     echo-server-service
Namespace:                default
Labels:                   app=echo-server
Annotations:              <none>
Selector:                 app=echo-server
Type:                     LoadBalancer
IP:                       10.107.107.166
Port:                     <unset>  8888/TCP
TargetPort:               8888/TCP
NodePort:                 <unset>  30915/TCP
Endpoints:                172.17.0.7:8888,172.17.0.8:8888
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```



如果我先在有i一个service的名字如上所示叫做 `echo-server-service` , 在k8s 中若要获取其 IP地址，只需要进行lookup即可。 

```
 kubectl get svc -n kube-system
```



```go
addresses, err := net.LookupHost("echo-server-service.default.svc.cluster.local")
```

DNS 记录 `echo-server-service.default.svc.cluster.local` 的含义是 ： `<service_name>.<namespace>.svc.<domain>  `

+ `echo-server-service` : service name
+ `default` : namespace 
+ svc 是固定的，标识是service component
+ cluster.local 是 domain 域名 ， 默认的

**另外如果 两个service 处于同一个 namespace 中，可以直接通过service 那么 进行访问**

**k8s 的DNS 并不是 round-robin DNS, 并不能起到负载均衡的作用**