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

