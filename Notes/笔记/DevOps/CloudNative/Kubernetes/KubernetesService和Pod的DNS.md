# Kubernetes之Service和Pod的DNS

## 介绍

Kubernetes DNS在集群上调度DNS Pod和服务，并配置`kubelet`以告知各个容器使用DNS服务的IP来解析DNS名称。

### 什么获得DNS名称?

集群中定义的每个Service（包括DNS服务器本身）都会分配一个DNS名称。默认情况下，客户端Pod的DNS搜索列表将包含Pod自己的`namespace`和集群的默认域。这通过示例得到最好的说明：

假设在Kubernetes `namespace`bar中名为foo的`Service`。在`namespace`bar中运行的Pod可以通过简单地foo执行DNS查询来查找此服务。在`namespace`quux中运行的Pod可以通过对foo.bar执行DNS查询来查找此服务。

## Service

### A records

`Normal`（不是headless）服务被分配了一个名为`my-svc.my-namespace.svc.cluster.local`形式的DNS A记录。这解析为服务的`Cluster IP`。

`Headless`（没有Cluster IP）服务还为`my-svc.my-namespace.svc.cluster.local`形式的名称分配了DNS A记录。与`Normal`不同，这将解析为服务选择的容器的IP集。客户端应该使用该集合，或者该集合中的标准循环选择。

### SRV records

为命名端口创建SRV记录，这些端口是`Normal`服务或`Headless`服务的一部分。对于每个命名端口，SRV记录的格式为`_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`。对常规服务，这将解析为端口号和域名：`my-svc.my-namespace.svc.cluster.local`。对于`headless`服务，这将解析为多个答案，一个用于支持服务的每个Pod，并包含`auto-generated-name.my-svc.my-namespace.svc.cluster.local`形式的Pod的端口号和域名。

## Pod

### Pod的主机名和子域字段

目前，当创建Pod时，其主机名是Pod的`metadata.name`值。

Pod的`spec`（规范）有一个可选`host`字段，可用于指定Pod的主机名。指定后，它优先于Pod的名称作为Pod的主机名。例如，如果Pod的`hostname`字段设置为`my-host`，则Pod的主机名将设置为`my-host`。

Pod的`spec`(规范)还有一个可选的`subdomain`字段，可用于指定其子域。例如，在`my-namespace`中，`hostname`设置为`foo`,`subdomain`设置为`bar`的Pod将具有完全限定域名（FQDN）`foo.bar.my-namespace.svc.cluster.local`。

例如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
```

