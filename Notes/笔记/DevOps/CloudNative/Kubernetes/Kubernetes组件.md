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

运行控制器的Master服务器上的组件。

逻辑上，每个控制器是一个单独的过程，但为了降低复杂性骂他们都被编译称为单个二进制文件并在单个进程中运行。

这些控制器包括：

-  Node Controller:负责在节点出现故障时通知和相应。
- Replication Controller:负责为系统中的每个`Replication Controller`对象维护正确的Pod数量。
- Endporints Controller:填充Endpoints对象，（即，连接Services&Pods）。
- Service Account & Token Controllers:为新的命名空间创建默认账户和API访问令牌。

### cloud-controller-manager

`cloud-controller-manager`运行与底层云提供商交互的控制器。云控制器管理器二进制文件是Kubernetes1.6版本中引入的alpha功能。

`cloud-controller-manager`仅运行特定于云提供程序的控制器循环。必须在`kube-controller-manager`中禁用这些控制器循环。可以通过在启动`kube-controller-manager`时将`--cloud-provider`标记设置为`external`来禁用控制器循环。

`cloud-controller-manager`允许云供应商代码和Kubernetes代码相互独立发展。在以前的版本中，核心Kubernetes代码依赖于特定于云提供程序的功能代码。在未来的版本中，云提供商特有的代码应由云供应商自己维护，并在运行Kubernetes时链接到`cloud-controller-manager`。

以下控制器具有云提供程序依赖项：

- Node Controller：用于检查云提供商，以确定节点在停止响应后是否已在云中删除。
- Route Controller：由于在底层云基础架构中设置路由。
- Service Controller：用于创建，更新和删除云提供商的负载均衡器。
- Volume Controller：用于创建，附加和装载卷，以及与云提供商交互以协调卷。



## Node组件

Node组件在每个Node运行，维护正在运行的Pod并提供Kubernetes运行时环境。

### kubelet

在集群中的每个节点上运行的代理。它确保容器在pod中运行。

`kubelet`采用通过各种机制提供的一组`PodSpecs`,并确保那些`PodSpecs`中描述的容器运行且正常。`kubelet`不管理不是由Kubernetes创建的容器。

### kube-proxy

`kube-proxy`通过维护主机上的网络规则并执行连接转发来启用Kubernetes服务抽象。

### Container Runtime

容器运行时是负责运行容器的软件。Kubernetes支持的多种运行时：Docker，rkt，runc和任何OCI运行时规范实现。



## Addons

插件是实现集群功能和Pod和Service。可以通过`Deployments`，`ReplicationControllers`等管理pod。NameSpaced插件对象在`kube-system`命名空间中创建。

选定的插件如下所述，

### DNS

虽然其他插件并非严格要求，但所有Kubernetes集群都应具有集群DNS，因为许多示例都依赖于它。

集群DNS是DNS服务器，除了环境中的其他DNS服务器外，它还为Kubernetes服务提供DNS记录。

Kubernetes启动的容器会在DNS搜索中自动包含此服务器。

### WebUI

Kubernetes集群基于web的管理UI。

### 容器资源监控

记录有关中央数据库中容器的通用时序指标，并提供用于预览该数据的UI。

### 集群级日志

将容器日志保存到具有搜索/浏览界面的中央日志存储。

