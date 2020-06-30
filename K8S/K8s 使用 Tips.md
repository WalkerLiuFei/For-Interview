# k8s 使用

## 通过(ExternalName `Service`)(无selector的Service访问外部服务)

希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。

- 希望服务指向另一个 [命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces) 中或其它集群中的服务。
- 您正在将工作负载迁移到 Kubernetes。 在评估该方法时，您仅在 Kubernetes 中运行一部分后端。

在任何这些场景中，都能够定义没有 selector 的 `Service`。 实例:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

由于此服务没有选择器，因此 *不会* 自动创建相应的 Endpoint 对象。 您可以通过手动添加 Endpoint 对象，将服务手动映射到运行该服务的网络地址和端口：

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

**因为 [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) 不支持将虚拟 IP 作为目标。 所以不能使用看k8s的集群IP作为目标**



![iptables代理模式下Service概览图](https://d33wubrfki0l68.cloudfront.net/27b2978647a8d7bdc2a96b213f0c0d3242ef9ce0/e8c9b/images/docs/services-iptables-overview.svg)