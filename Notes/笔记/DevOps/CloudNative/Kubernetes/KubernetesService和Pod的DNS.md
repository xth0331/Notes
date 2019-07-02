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

如果在与Pod相同的`namespace`中存在`headless`且与子域名称相同，则集群的KubeDNS服务器也会返回Pod的完全限定域名主机的A记录。例如，给主机名设置为`busybox-1`且子域设置为`default-subdomain`的Pod，以及同一namespace中名为`default-subdomain`的`headless`服务，该Pod将看到自己的FQDN为`busybox-1.default-sudomain.my-namespace.svc.cluster.local`。DNS以该名称提供A记录，指向Pod的IP。Pod `busybox1`和`busybox2`都可以拥有不同的A记录。

`Endpoint`对象可以指定任何端点地址的主机名以及IP。

> 由于未为Pod名称创建A记录，因此要创建Pod的A记录需要`hostname`。没有`hostname`但具有`subdomain`的Pod会为`headless`服务创建A记录，执行Pod的IP地址。此外，除非在服务上设置了`publishNotReadyAddresses=True`，否则Pod需要准备好才能拥有记录。

### Pod DNS 策略

可以基于每个Pod设置DNS策略。目前，Kubernetes支持以下特定于Pod的DNS策略。这些策略在Pod `spec`的`dnsPolicy`字段中指定。

- `Default`：Pod从运行Pod的节点继承解析配置。
- `ClusterFirst`：任何与配置的集群域后缀不匹配的DNS查询都会转发到从该节点集成的上游DNS服务器。集群管理员可能配置了额外的`stub-domain`和上游DNS服务器。
- `ClusterFirstWithHostNet`：对于使用`hostNetwork`运行的Pod。应该明确设置其DNS策略。
- `None`：它允许Pod忽略Kubernetes环境中的DNS设置。应该使用Pod中`spec.dnsConfig`字段提供所有DNS设置。

> `Default`不是默认的DNS策略，如果未明确指定`dnsPolicy`，则使用`ClusterFirst`。

下面的示例显示了一个Pod，其DNS策略设置为`ClusterFirstWithHostNet`，因为它将`hostNetwork`设置为`true`。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```

### Pod DNS 配置

Pod的DNS配置允许用户更多地控制Pod的DNS设置。

`dnsConfig`字段是可选的，可以使用任何`dnsPolicy`设置。但是，当Pod的`dnsPolicy`设置为“ `None`”时，必须指定`dnsConfig`字段。

以下是用户可在`dnsConfig`字段中指定的属性：

- `nameservers`：将用作Pod的DNS服务器的IP地址列表。最多可以指定3个IP地址。当Pod `dnsPolicy` 设置为“ `None`”时，列表必须至少包含一个IP地址，否则此属性是可选的。列出的服务器将合并到从指定的DNS策略生成的基本名称服务器，并删除重复的地址。
- `searches`：Pod中主机名查找的DNS搜索域列表。此属性是可选的。指定后，提供的列表将合并到从所选DNS策略生成的基本搜索域名中。删除重复的域名。Kubernetes最多允许6个搜索域。
- `options`：可选的对象列表，其中每个对象可以具有`name` 属性（必需）和`value`属性（可选）。此属性中的内容将合并到从指定的DNS策略生成的选项中。删除重复的条目。

以下是具有自定义DNS设置的Pod示例：

```yaml
`apiVersion: v1 kind: Pod metadata:   namespace: default   name: dns-example spec:   containers:     - name: test       image: nginx   dnsPolicy: "None"   dnsConfig:     nameservers:       - 1.2.3.4     searches:       - ns1.svc.cluster.local       - my.dns.search.suffix     options:       - name: ndots         value: "2"       - name: edns0 `
```

创建上面的Pod时，容器`test`在其`/etc/resolv.conf`文件中获取以下内容：

```bash
nameserver 1.2.3.4
search ns1.svc.cluster.local my.dns.search.suffix
options ndots:2 edns0
```

对于IPv6设置，搜索路径和名称服务器应设置如下：

```bash
kubectl exec -it dns-example -- cat /etc/resolv.conf
nameserver fd00:79:30::a
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

