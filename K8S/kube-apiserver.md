# API Server

kube-apiserver 是 Kubernetes 最重要的核心组件之一，主要提供以下的功能

- 提供集群管理的 REST API 接口，包括认证授权、数据校验以及集群状态变更等

- 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才直接操作 etcd）

  

在实际使用中，通常通过 [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 来访问 apiserver，也可以通过 Kubernetes 各个语言的 client 库来访问 apiserver。在使用 kubectl 时，打开调试日志也可以看到每个 API 调用的格式







## API 访问

有多种方式可以访问 Kubernetes 提供的 REST API：

- [kubectl](https://feisky.gitbooks.io/kubernetes/components/kubectl.html) 命令行工具
- SDK，支持多种语言
  - [Go](https://github.com/kubernetes/client-go)