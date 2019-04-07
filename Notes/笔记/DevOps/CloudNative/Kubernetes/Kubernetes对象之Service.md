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

![](https://blog-image.nos-eastchina1.126.net/Service-iptables.jpg)

在任何这些代理模式中，绑定到`Service`的IP:Port的任何流量都代理到了适当的后端，而客户端不知道有关Kubernetes或`Service`或Pod的任何信息。可以通过将`service.spec.sessionAffinity`设置为`ClientIP`（默认为“None”）来选择基于客户端IP的会话亲和关系，并且可以通过设置字段`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`来设置最大会话黏连时间。如果已将`service.spec.sessionAffinity`设置为`ClientIP`，则timeout（默认为10800）。

## Multi-Port Services

