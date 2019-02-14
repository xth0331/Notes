# Kubernetes对象之namespaces



Kubernetes支持由同一物理集群支持的多个虚拟集群。这些虚拟集群成为`namespaces`，

## 何时使用多个namespaces

namespaces旨在用于多个用户分布在多个团队或项目中的环境中。对于具有几个到几十个用户的集群，根本不需要创建或考虑namespaces，当需要它们提供的功能时，请开始使用命名空间。

namespaces提供名称范围。资源名称在namespaces中必须是唯一的，而不是跨namespaces。

namespaces是一种在多个用户之间划分集群资源的方法（通过资源配额）。

在未来的Kubernetes版本中，默认情况下，同`namespaces`中的对象将具有相同的访问控制策略。

没有必要使用多个`namespaces`来分隔略有不同的资源，例如同一软件的不同版本：使用标签来区分同一`namespaces`中的资源。



##使用namespaces



