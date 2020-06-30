# kube-scheduler

1. kube-scheduler 负责分配调度 Pod 到集群内的节点上，它监听 kube-apiserver，查询还未分配 Node 的 Pod，然后根据调度策略为这些 Pod 分配节点（更新 Pod 的 `NodeName` 字段）。

2. [kube-scheduler](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-scheduler/) 是 Kubernetes 集群的默认调度器，并且是集群 [控制面](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-control-plane) 的一部分。如果你真的希望或者有这方面的需求，kube-scheduler 在设计上是允许你自己写一个调度组件并替换原有的 kube-scheduler。
3. 在做调度决定时需要考虑的因素包括：
   1. 单独和整体的资源请求
   2. 硬件/软件/策略限制
   3. 亲和以及反亲和要求
   4. 数据局域性
   5. 负载间的干扰等等。



有三种方式指定 Pod 只运行在指定的 Node 节点上

- nodeSelector：只调度到匹配指定 label 的 Node 上
- nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作
- podAffinity：调度到满足条件的 Pod 所在的 Node 上

[将pod分配给节点](https://kubernetes.io/zh/docs/concepts/configuration/assign-pod-node/)

1. 显式的用 node selector 将pod 部署到 带指定label的机器上



## kube-scheduler 调度流程

kube scheduler 调度参考 ：[kube schedule 调度器](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/scheduling-framework/)

kube-scheduler 给一个 pod 做调度选择包含两个步骤：

1. 过滤 ： 过滤阶段会将所有满足 Pod 调度需求的 Node 选出来。例如，PodFitsResources 过滤函数会检查候选 Node 的可用资源能否满足 Pod 的资源请求。在过滤之后，得出一个 Node 列表，里面包含了所有可调度节点；通常情况下，这个 Node 列表包含不止一个 Node。如果这个列表是空的，代表这个 Pod 不可调度。
2. 打分： 在打分阶段，调度器会为 Pod 从所有可调度节点中选取一个最合适的 Node。根据当前启用的打分规则，调度器会给每一个可调度节点进行打分。

一个调度示例 ： 

```
For given pod:

    +---------------------------------------------+
    |               Schedulable nodes:            |
    |                                             |
    | +--------+    +--------+      +--------+    |
    | | node 1 |    | node 2 |      | node 3 |    |
    | +--------+    +--------+      +--------+    |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+

    Pred. filters: node 3 doesn't have enough resource

    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+
    |             remaining nodes:                |
    |   +--------+                 +--------+     |
    |   | node 1 |                 | node 2 |     |
    |   +--------+                 +--------+     |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+

    Priority function:    node 1: p=2
                          node 2: p=5

    +-------------------+-------------------------+
                        |
                        |
                        v
            select max{node priority} = node 2
```





## namespace 和 资源配置

[ResourceQuotas](https://kubernetes.io/zh/docs/concepts/policy/resource-quotas/)

在创建 namespace时可以为其分配资源配置，例如：

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
```

## Taints 和 tolerations

[Taints And tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

Taints 和 tolerations 用于保证 Pod 不被调度到不合适的 Node 上，其中 Taint 应用于 Node 上，而 toleration 则应用于 Pod 上。

目前支持的 taint 类型

- NoSchedule：新的 Pod 不调度到该 Node 上，不影响正在运行的 Pod
- PreferNoSchedule：soft 版的 NoSchedule，尽量不调度到该 Node 上
- NoExecute：新的 Pod 不调度到该 Node 上，并且删除（evict）已在运行的 Pod。Pod 可以增加一个时间（tolerationSeconds），

## 参考

- [Pod Priority and Preemption](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/)
- [Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)
- [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)
- [Advanced Scheduling in Kubernetes](https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/)