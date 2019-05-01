# Kubernetes之Service和Pod的DNS

## 介绍

Kubernetes DNS在集群上调度DNS Pod和服务，并配置`kubelet`以告知各个容器使用DNS服务的IP来解析DNS名称。

### 什么获得DNS名称?

集群中定义的每个Service（包括DNS服务器本身）都会分配一个DNS名称。默认情况下，客户端Pod的DNS搜索列表将包含Pod自己的`namespace`和集群的默认域。这通过示例得到最好的说明：

假设在Kubernetes `namespace`bar中名为foo的`Service`。在`namespace`bar中运行的Pod可以通过简单地foo执行DNS查询来查找此服务。在`namespace`quux中运行的Pod可以通过对foo.bar执行DNS查询来查找此服务。

## Service

### A records

