# K8s 中的负载均衡实现方式

通过 LoaderBalancer, 在外部访问时一般通过云厂商的LB访问 K8s集群中的Service, 然后Service下面才是具体的Pod。 



K8s中服务发现时通过DNS 进行的。