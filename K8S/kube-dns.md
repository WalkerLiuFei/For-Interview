# DNS

DNS 是 Kubernetes 的核心功能之一，通过 kube-dns 或 CoreDNS 作为集群的必备扩展来提供命名服务。+



##  支持的DNS格式

- Service

  - A record：生成

    ```
    my-svc.my-namespace.svc.cluster.local
    ```

    ，解析 IP 分为两种情况

    - 普通 Service 解析为 Cluster IP
    - Headless Service 解析为指定的 Pod IP 列表

  - SRV record：生成 `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`

- Pod

  - A record：`pod-ip-address.my-namespace.pod.cluster.local`
  - 指定 hostname 和 subdomain：`hostname.custom-subdomain.default.svc.cluster.local`，如下所示



```
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```



### Pod 的 DNS 设定

Pod 的 DNS 配置可让用户对 Pod 的 DNS 设置进行更多控制。

`dnsConfig` 字段是可选的，它可以与任何 `dnsPolicy` 设置一起使用。 但是，当 Pod 的 `dnsPolicy` 设置为 “`None`” 时，必须指定 `dnsConfig` 字段。

用户可以在 `dnsConfig` 字段中指定以下属性：

- `nameservers`: 将用作于 Pod 的 DNS 服务器的 IP 地址列表。最多可以指定3个 IP 地址。 当 Pod 的 `dnsPolicy` 设置为 “`None`” 时，列表必须至少包含一个IP地址，否则此属性是可选的。列出的服务器将合并到从指定的 DNS 策略生成的基本名称服务器，并删除重复的地址。
- `searches`: 用于在 Pod 中查找主机名的 DNS 搜索域的列表。此属性是可选的。指定后，提供的列表将合并到根据所选 DNS 策略生成的基本搜索域名中。 重复的域名将被删除。   Kubernetes最多允许6个搜索域。
- `options`: 对象的可选列表，其中每个对象可能具有 `name` 属性（必需）和 `value` 属性（可选）。 此属性中的内容将合并到从指定的 DNS 策略生成的选项。 重复的条目将被删除。

以下是具有自定义DNS设置的Pod示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

创建上面的Pod后，容器 `test` 会在其 `/etc/resolv.conf` 文件中获取以下内容：

```reStructuredText
nameserver 1.2.3.4
search ns1.svc.cluster-domain.example my.dns.search.suffix
options ndots:2 edns0
```

## 支持配置私有 DNS 服务器和上游 DNS 服务器

从 Kubernetes 1.6 开始，可以通过为 kube-dns 提供 ConfigMap 来实现对存根域以及上游名称服务器的自定义指定。例如，下面的配置插入了一个单独的私有根 DNS 服务器和两个上游 DNS 服务器。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
data:
  stubDomains: |
    {“acme.local”: [“1.2.3.4”]}
  upstreamNameservers: |
    [“8.8.8.8”, “8.8.4.4”]
```

### kube-dns 工作原理

如下图所示，kube-dns 由三个容器构成：

- kube-dns：DNS 服务的核心组件，主要由 KubeDNS 和 SkyDNS 组成
  - KubeDNS 负责监听 Service 和 Endpoint 的变化情况，并将相关的信息更新到 SkyDNS 中
  - SkyDNS 负责 DNS 解析，监听在 10053 端口 (tcp/udp)，同时也监听在 10055 端口提供 metrics
  - kube-dns 还监听了 8081 端口，以供健康检查使用
- dnsmasq-nanny：负责启动 dnsmasq，并在配置发生变化时重启 dnsmasq
  - dnsmasq 的 upstream 为 SkyDNS，即集群内部的 DNS 解析由 SkyDNS 负责
- sidecar：负责健康检查和提供 DNS metrics（监听在 10054 端口）

![img](https://feisky.gitbooks.io/kubernetes/components/images/kube-dns.png)