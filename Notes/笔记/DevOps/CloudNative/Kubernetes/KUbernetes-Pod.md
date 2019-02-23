# Kubernetes之Pod

## 认识Pods

`pod`是Kubernetes的基本构建单元——是创建或部署Kubernetes对象模型中最小最简单的单元。一个`pod`表示集群上正在运行的进程。

一个`pod`包含一个应用容器（或者，在某些情况下，是多个容器）， 存储资源，一个独立网络IP，以及控制容器运行方式的选项。一个`pod`表示`deployment`单元：Kubernetes中的单个应用程序实例，可能由单个容器或少量紧耦合并共享资源的容器组成。

Docker是Kubernetes Pod中最常用的容器运行时，但Pod也支持其他容器运行时。

Kubernetes及群众Pod可以以两种主要方式使用：

- **运行单个容器Pod**

  *one-container-per-Pod*模型是最常见的；在这种情况下，可以将Pod视为单个容器的包装，而Kubernetes直接管理Pod而不是容器本身。

  