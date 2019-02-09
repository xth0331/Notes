# Kubernetes组件

## Master组件

Master组件提供集群的控制平面。Master组件做出关于集群的全局决策（例如，调度），以及检测和响应集群事件（当复制控制器的‘副本’字段不满足时启动新的Pod）。

Master组件可以在集群中的任何计算机上运行。但是为了简单起见，设置脚本通常会在同一台计算机上启动所有Master组件，并且不在此计算机上运行用户容器。

### kube-apiserver

暴露Kubernetes API的master组件。它是Kubernetes控制平面的前端。它旨在水平扩展-也就是说，它通过部署更多实例来扩展。

### etcd

一致且高度可用的键值存储，用作Kubernetes的所有集群数据的后端存储。

始终为Kubernetes集群提供etcd数据的备份计划。

### kuber-scheduler

Master服务器上的组件，用于监控未创建节点和新创建的Pod，并选择一个节点供其运行。

调度决策所考虑的因素包括个人和集群资源需求，硬件/软件/策略约束，亲和力和反亲和性规范，数据位置，工作负载间干扰和最后限期。

### kube-controller-manager



### cloud-controller-manager







