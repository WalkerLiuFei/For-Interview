# 零（环境准备）

Mac

minikube

1. 使用 brew 直接安装minikube
2. 然后使用 minikube start , 这个时候 会使用虚拟机安装一个 docker, 这个docker 默认是作为了 minikube cluster的 node机器，而宿主机，就是使用的MAC 电脑 是作为了master
3. docker 里面的

# 一

然后执行对应命令。

deploy.yaml 是deployment， deployment 是管理 pod的命令等，可以在里面限制内存使用大小，运行命令等等。

1. 阅读build.sh 脚本中的命令，那些命令

# 二

对于 `deployment.yaml` 配置的简单说明

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-server-deployment
  labels: ## labels 没什么特别意义
    app: echo-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: echo-server
  template:
    metadata:
      labels:
        app: echo-server
    spec:
      containers:
        - name: echo-server
          image: walkerliu/echo-server-example
          ports:
            - containerPort: 8888 
```

1. kind :  # what kind of object you want to create,[Deployment,Service,Replica Set,Replication Controller, StateFulSets,DaemonSet,...] 
2. `selector` 字段定义 Deployment 如何查找要管理的 Pods。 在这种情况下，只需选择在 Pod 模板（`app: nginx`）中定义的标签。但是，更复杂的选择规则是可能的，只要 Pod 模板本身满足规则。
3. `template` 字段包含以下子字段：
   1. Pod 标记为`app: nginx`，使用`labels`字段
   2. Pod 模板规范或 `.template.spec` 字段指示 Pods 运行一个容器， `nginx`，运行 `nginx` [Docker Hub](https://hub.docker.com/)版本1.7.9的镜像 。
   3. 创建一个容器并使用`name`字段将其命名为 `nginx`。
4. 





