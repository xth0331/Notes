# 了解Kubernetes对象

Kubernetes对象是Kubernetes系统中的持久实体。Kubernetes使用这些实体来表示集群的状态。具体来说，他们可以描述：

- 正在运行的容器化应用程序（以及在那些节点上）
- 这些应用程序可用的资源
- 有关这些应用程序行为方式的策略，例如重新启动策略，升级和容错。

Kubernetes对象是一个“record of intent(意图记录)”-你创建对象，Kubernetes系统将不断努力确保对象存在。通过创建一个对象，可以有效地告诉Kubernetes系统希望集群的工作负载看起来像什么；这是您的集群**期望状态**。

操作Kubernetes对象 —— 五落实创建、修改、或者删除 —— 需要使用Kubernetes API。比如，当使用`kubectl`命令行时，CLI会执行必要的Kubernetes API调用，也可以在程序中直接调用Kubernetes API。

对象 

