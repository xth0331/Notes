# Kubernetes对象之Service

Kubernetes的Pod是会终止的。他们没有复活。`ReplicaSets`特别是动态的创建和销毁Pod（例如横向拓展）。虽然每个Pod都有自己的IP地址，但即使这样也不能依赖它们保持稳定。这会导致一个问题：如果某些Pod在Kubernetes集群内想其他人提供功能，如何找出并跟踪该集合中的那些Pod？

通过**Services**。

Kubernetes `Service`是一个抽象，它定义了一组逻辑Pod和一个访问他们的策略 - 有时也成为微服务。`Service`所针对的Pod集合通常由`Label Selector`（标签选择器）。

例如，考虑一个运行3个副本的image处理后端。这些副本是可替换的——前端并不关系他们使用哪个后端。虽然组成后端集的实际Pod可能会发生更改，但前端不应该意识到这一点，也不应该改自己跟踪后端列表。`Service`抽象实现了这种解耦。

对于Kubernetes原生应用，Kubernetes提供了一个简单的`Endpoints`API，只要`Service`中的Pod发生变化，它就会更新。对于非原生应用，Kubernetes提供一个基于虚拟IP的服务桥接，重定向到后端Pod。

## 定义service

Kubernetes中`Service`是一个REST对象，类似与Pod。与所有REST对象一样，可以将`Service`定义POSTed到apiserver来创建一个新实例。例如，假设有一组Pod，每个Pod都暴露端口9376并带有标“app=MyApp”。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  posts:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

此规范将创建一个名为**my-service**的新`Service`对象，该对象使用`app=MyApp`标签的Pod上的TCP端口9376。此`Service`还分配一个IP地址（ClusterIP），由服务代理使用。`Service`的选择器将持续评估，并将结果POSTed到名为`my-service`的`Endpoints`对象。

注意，`Service`可以将传入端口映射到任何`targetPort`。默认情况下，`targetPort`将设置为与端口字段相同的值。也许更有趣的是，`targetPort`可以是一个字符串，指的是后端Pod中端口的名字。分配给该名称的实际端口号在每个后端Pod中可以不同。这位部署和发展`Service`提供了很大的灵活性。例如，可以更改Pod在下个版本的后端软件中公开的端口号，而不会破坏客户端。

`TCP`是`Service`的默认协议，还可以使用任何其他支持的协议。由于许多`Service`需要暴露多个端口，因此Kubernetes支持`Service`对象上定义多个端口。每个端口定义可以具有相同或不同的协议。

### Services without selectors

`Service`通常抽象访问Kubernetes的Pod，但它们也可以抽象其他类型的后端。例如:

- 希望在生产中拥有外部数据库集群，但在测试中使用自己的数据库。
- 希望将`Service`指向另一个namespace或拎一个集群上的`Service`。
- 正在将工作负载迁移到Kubernetes，并且一些后端在Kubernetes之外运行。

在任何这些场景中，都可以定义不带选择器（selector）的`Service`:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

由于此`Service`没有选择器，因此不会创建相应的`Endpoints`对象。可以手动将`Service`映射到自己的特定endpoint：

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
    - ip: 1.2.3.4
  ports:
    - port:9376 
```

> 注意：endpoint IP可能不是回环，本地链路或本地多播链路。他们不能是其他Kubernetes `Service`的ClusterIP，因为`kube-porxy`组件不支持虚拟Ip作为目标。

在没有选择器的情况下访问Service的工作方式与具有选择器的方式相同。流量将路由到用户定义的endpoint（例子中的1.2.3.4:9376）。

 `ExternalName`是一种特殊情况的`Service`。他没有选择器而是使用DNS名称。

## Virtual IPs and service proxies

Kubernetes集群中的每个节点都运行一个`kube-proxy`。`kube-proxy`负责为`ExternalName`以外的类型的服务实现一种形式的虚拟IP。

在Kubernetes v1.0中，`Service`是四层（基于IP的TCP/UDP），proxy仅仅在用户空间中。在v1.1中，添加了`Ingress`API来表示七层（HTTP）服务，也添加了iptables代理，并成为v1.2依赖的默认工作模式。v1.8添加了ipvs代理。

### 代理模式: 用户空间

在此模式下,`kube-proxy`灰监视master以添加和删除`Service`和`Endpoints`对象。对于每个`Service`，它在本地节点打开一个端口（随机选择）。与此代理端口的任何连接都将代理到其中一个`Service`的后端Pod。根据`Service`的`SessionAffinity`决定使用哪个后端Pod。最后，它安装iptables规则，捕获流量到`Service`的ClusterIP（虚拟）和端口，并将流量重定向到代理后端Pod的代理端口。默认情况下，后端的选择是循环。

![](https://blog-image.nos-eastchina1.126.net/Service-userspace.jpg)

### 代理模式: iptables

在此模式下，`kube-proxy`会监视master已添加和删除`Service`的`Endpoints`对象。对于每个`Service`，它安装iptables规则，捕获到`Service`的ClusterIP（虚拟）和端口的流量，并将流量重定向到`Service`后端之一。对于每个`Endpoints`对象，它会安装选择后端Pod的iptables规则。默认情况下，后端选择是随机的。

显然，iptables不需要在用户空间和内核空间之间切换，它应该比用户空间代理更快，更可靠。但是，与用户空间代理不同，如果最初选择的那个Pod没有响应，则iptables代理无法自动重试另一个Pod，因为依赖于具有工作准备情况的探针。

![](https://blog-image.nos-eastchina1.126.net/Service-ip.jpg)



### 代理模式: ipvs

在此模式下，kube-proxy监控Kubernetes Service和Endpoint，调用`netlink`接口以相应地创建ipvs规则并定期与Kubernetes Service和 Endpoint同步ipvs规则，以确保ipvs状态与期望一致。访问Servcie时，流量将被重定向到其中的一个后端Pod。

与iptables类似，ipvs基于netfilter钩子函数，但使用哈希表作为底层数据结构并在内核空间工作。这意味着ipvs可以更快的重定向流量，并且在同步代理规则是具有更好的性能。此外，ipvs为负载均衡算法提供更多选项，例如：

- `rr`: round-robin
- `lc`: least connection
- `dh`: destination hashing
- `sh`: source hashing
- `sed`: shortest expected delay
- `nq`: never queue

> ipvs模式假设在运行kube-proxy之前在节点上启用了ipvs内核模块。当kube-proxy以ipvs代理模式启动时，kube-proxy将验证节点上是否启用了ipvs模块，如果未启用，则kube-proxy将回退到iptables模式。

![](https://blog-image.nos-eastchina1.126.net/Service-iptables.jpg)

在任何这些代理模式中，绑定到`Service`的IP:Port的任何流量都代理到了适当的后端，而客户端不知道有关Kubernetes或`Service`或Pod的任何信息。可以通过将`service.spec.sessionAffinity`设置为`ClientIP`（默认为“None”）来选择基于客户端IP的会话亲和关系，并且可以通过设置字段`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`来设置最大会话黏连时间。如果已将`service.spec.sessionAffinity`设置为`ClientIP`，则timeout（默认为10800）。

## Multi-Port Services

许多Service需要暴露一个以上端口。对于这种情况，Kubernetes支持Service对象上的多个端口定义。使用多个端口时，必须提供所有端口名称，以便可以消除端口歧义。例如：

```yaml
kind: Service
apiVersion: v1
metadata: 
  name: my-service
spec:
  selector: 
    app: Myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```

注意，端口名称只能包含小写字母，数字和“-”，并且必须以小写字母或数字开头和结尾。

## 选择自己的IP地址

可以将自己的集群IP地址指定为Service创建请求的一部分。为此，设置`.spec.clusterIP`字段。例如，如果已经有一个希望重用的DNS条目，或者为特定IP地址配置并且难以重新配置的旧系统。用户选择的IP地址必须是有效的IP地址，并且在由API服务器指定的`service-cluster-ip-range`CIDR范围内。如果IP地址无效则apiserver返回422状态码以表示该值无效。

### 为什么不适用 round-robin DNS?

一个经常出现的问题是我们为什么用虚拟IP做所有这些事情而不仅仅是使用标准的循环DNS：

- DNS库的历史悠久，不尊重DNS TTL并缓存查询的结果。
- 许多应用执行DNS查找以此并缓存结果。
- 即使应用和库进行了适当的重新解析，每个客户端反复重新解析DNS的负载也难以管理。

## 发现服务

Kubernetes支持两种发现`Service`的主要模式 - 环境变量和DNS。

### 环境变量

当Pod在节点上运行时，kubelet为每个活动Service添加一组环境变量。它支持Docker links和更简单的`{SVCNAME}_SERVICE_HOST`和`{SVCNAME}_SERVICE_PORT`变量，其中Service名称为大写，“-” 转换为“_”。

例如，暴露TCP端口6379并已分配集群IP地址10.0.0.11的Service`redis-master`生成一下环境变量：

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

这确实意味着要求 - 必须在Pod本身创建Pod想要访问的任何服务，否则将不会填充环境变量。DNS没有此限制。

### DNS

可选（尽管强烈推荐）集群加载项是DNS服务器。DNS服务器监控Kubernetes API以获取新Service，并为每个Service创建一组DNS记录。如果在整个集群中启用了DNS，则所有Pod应该能够自动对Service进行解析。

例如，如果在名为`my-ns`的Kubernetes命名空间中有一个名为`my-service`的服务，则会创建`my-service.my-ns`的DNS记录。存在于`my-ns`命名空间中的Pod应该能够通过简单地对`my-service`进行名称查找来找到它。存在于其他命名空间的Pod必须将名称限定为`my-service.my-ns`。这些名称查询的结果第ClusterIP。

Kubernetes还支持命名端口的DNS SRV记录。如果`my-service.my-ns`服务具有带协议TCP的名为`http`的端口，则可以对`_http._tcp.my-service.my-ns`执行DNS SRV查询一发现端口号`http`。

Kubernetes DNS服务器是访问`ExternalName`类型服务的唯一方法。

## Headless services

有时不需要或不需要LB和单个Service IP。在这种情况下，可以通过为集群IP(`.spec.clusterIP`)指定`None`来创建无头服务。

此选项允许开发人员通过允许他们自由地以自己的方式进行发现来减少与Kubernetes系统的耦合。应用程序仍然可以使用自注册模式，并且可以轻松的在此API上构建适用于其他发现系统的适配器。

对于此类服务，未分配 Cluster IP，`kube-proxy`不处理这些服务，并且平台没有为他们执行负载均衡代理。如何自动配置DNS取决于服务是否定义了选择器。

### 有selector

对于定义选择器的headless服务，`Endpoint`控制器在API中创建`Endpoint`记录，并修改DNS配置以返回直接指向支持`Service`的Pod的A记录（地址）。



### 没有 selector

对于未定义选择器的headless服务，`Endpoint`控制器不会创建`Endpoint`记录，但是，DNS系统会查找并配置：

- `ExternalName`类型服务的CNAME记录。
- 所有其他类型的与服务共享名称的任何`Endpoint`的记录。

## 发布服务 - service types

对于应用程序的某些部分（例如前端），可能希望将服务公开给外部（集群外）IP地址。

Kubernetes `ServiceTypes`允许指定所需的服务类型。默认为`ClusterIP`。

`Type`值机器行为是：

- `ClusterIP`：在集群内部IP上公开服务。选择此值使服务之能从集群中访问。它是默认的`ServiceType`。
- `NodePort`：在每个节点的IP的静态端口上公开服务。将自动创建`NodePort`服务将路由到的`ClusterIP`。可以通过请求`NodeIP:Node:Port`来从集群外部链接`NodePort`服务。
- `LoadBalancer`：使用云提供商的负载均衡器在外部公开服务。将自动创建外部负载均衡器将路由到的NodePort和ClusterIP服务。
- `ExternalName`：将服务映射到`ExternalName`字段的内容（例如foo.var.example.com），通过返回带有其值的`CNAME`记录。没有设置任何类型的代理。

### NodePort

如果将类型字段设置为`NodePort`，Kubernetes Master将从`--service-node-port-range`标志指定的范围（默认：30000-32767）分配端口，并且每个节点将代理该端口（相同端口）进入`Service`。该端口将在`Service`的`.spec.ports[*].nodePort`字段中报告。

如果要指定代理端口的特定IP，可以将`kube-proxy`中的`--nodeport-addresses`标志设置为特定的IP块。以逗号分隔的IP块列表（10.0.0.0/8,1.2.3.4/32）用于过滤此节点的本地地址。例如，如果使用标志`--nodeport-addresses=127.0.0.0/8`，kube-proxy将仅为`Node-Port`服务选择loopback接口。`--nodeport-addresses`默认为空，这意味着选择所有可用的接口并符合当前的`NodePort`行为。

如果需要特定的端口，可以在`nodePort`字段中指定一个值，系统将分配该端口，否则API事务将失败（即使需要自己处理可能的端口冲突）。指定的值必须在节点端口范围内。

这使开发人员可以自由地设置自己的负载均衡器，配置Kubernetes不完全支持的环境，甚至直接暴露一个或多个节点的IP地址。

注意，此服务将同时显示为`NodeIP:sepc.ports[*].nodePort`和`.spec.clusterIP:spec.ports[*].port`。（如果设置了kube-porxy中的`--nodeport-addresses`标志，则过滤NodeIP(s)）。

### LoadBalancer

在支持外部负载均衡器的云供应商上，将类型字段设置为`LoadBalancer`将为服务配置负载均衡器。负载均衡器的时间创建是异步发生的，有关配置的均衡器的信息将发布在`Service`的`.status.loadBalancer`字段中。例如：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 146.148.47.155
```

来自外部负载均衡器的流量将指向后端Pod，具体如何运作取决于云供应商。某些云提供程序允许指定`loadBalancerIP`。这种情况下，将使用用户指定的`loadBalancerIP`。如果未指定`loadBalancerIP`字段，则会将临时IP分配给`loadBalancer`。如果指定了`loadBalancerIP`但云提供程序不支持该功能，则字段将被忽略。

#### Internal load balancer

在混合环境中，又是需要从同一VPC内的服务路由流量。

在水平分割DNS环境中，需要两个`service`才能将外部和内部流量路由到`endpoint`。

#### SSL support on AWS

对在AWS上运行的集群上的部分SSL支持，从1.3K开始可以将第三个注释添加到`LoadBalancer`服务：

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

第一个指定要使用的证书的ARN。他可以是上载到IAM的第三方颁发者的证书，也可以是AWS Certificate Manager中创建的证书。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

第二个注释指定了Pod所用的协议。对于HTTPS和SSL，ELB将期望Pod通过加密连接进行身份验证。

HTTP和HTTPS将选择7层代理：