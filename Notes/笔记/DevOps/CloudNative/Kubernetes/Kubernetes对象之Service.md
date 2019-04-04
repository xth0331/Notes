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

此规范将创建一个名为**my-service**的新`Service`对象，该对象使用`app=MyApp`标签的Pod上的TCP端口9376。

