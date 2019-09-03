# Kubernetes Ingress

管理对集群中的Service的外部访问的API对象，通常是HTTP。

`Ingress`可以提供负载均衡，SSL终止和基于名称的虚拟主机。



## 术语

为清楚期间，本指南定义了以下术语：

| 术语                        | 解释                                                         |
| --------------------------- | ------------------------------------------------------------ |
| Node（节点）                | Kubernetes中的工作节点，是集群中的一部分。                   |
| Cluster（集群）             | 一组运行由Kubernetes管理的容器化应用程序的节点               |
| Edge router（边界路由）     | 为您的集群强制执行防火墙策略的路由器。这可以是由云提供商提供的，也可以是物理硬件。 |
| Cluster network（集群网络） | 一组逻辑或物理链接，根据Kubernetes网络模型促进群集内的通信。 |
| Service（服务）             | Kubernetes服务，使用标签选择器标识一组Pod。 除非另有说明，否则假定服务具有仅在群集网络内可路由的虚拟IP。 |



## 什么是Ingress

`Ingress`将HTTP和HTTPS路由从集群外部公开到集群内的服务。流量路由由入口资源上定义的规则控制。

```bash
   internet
           |
   [ Ingress ]
    --|-----|--
   [ Services ]
```

可以将`Ingress`配置为为服务提供外部可访问的URL，负载均衡流量，终止`SSL/TLS`以及提供基于名称的虚拟主机。一个`Ingress`控制器负责完成`Ingress`，通常有一个负载均衡器，虽然它也可以配置您的边界路由或额外的前端来处理流量。

`Ingress`不会暴露任意端口或协议。将除HTTP或HTTPS之外的服务暴露给互联网通常使用`Service.Type = NodePort`或`Service.Type = LoadBalancer`。

## 先决条件

必须有一个`Ingress`控制器来满足一个`Ingress`。仅创建Ingress资源无效。

您可能需要部署`ingress`控制器，例如`ingress-nginx`。也可以从许多`Ingress`控制器中选择。

理想情况下，所有`Ingress`控制器都应符合参考规范。实际上，各种`Ingress`控制器的运行方式略有不同。

> 注意： 请务必查看`Ingress`控制器的文档。

## Ingress 资源

一个最小的`Ingress`资源示例：

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        backend:
          serviceName: test
          servicePort: 80
```

与所有其他Kubernetes资源一样，一个`Ingress`需要`apiVersion`,`kind`和`metadata`字段。`Ingress`经常使用注释来配置一些选项，具体取决于`Ingress controller`，其中一个例子是`rewrite-target `。不通的`Ingress controller`支持不同的注释。

`Ingress`规范（spec）包含配置负载均衡或代理服务器的所有信息。最重要的是，它包含一个针对所有传入请求匹配的规则列表。`Ingress`资源仅支持HTTP流量的规则。

### Ingress 规则

每个HTTP规则都包含一下信息：

- 可选主机。在此示例中，未指定主机，因此该规则使用与通过指定的IP地址的所有入站HTTP流量。如果提供了主机（例如，foo.bar.com），则规则适用于该主机。
- 路径列表（例如 /testpath），每个路径都有一个用`serviceName`和定义的关联后端的`servicePort`。在负载均衡器将流量定向到引用的`service`之前，主机和路径都必须与传入请求的内容匹配。
- 后端是`service`和端口名称的组合。向`Ingress`发出的与主机和规则路径匹配的HTTP（和HTTPS）请求将发送到列出的后端。

默认后端通常在`Ingress`控制器中配置，以便为与规范中的路径不匹配的任何请求提供服务。



### 默认后端

没有规则的`Ingress`将所有流量发送到单个默认后端。默认后端通常是`Ingress controller`的配置选项，并且未在`Ingress`资源中指定。

如果没有任何主机或路径与`Ingress`对象中的HTTP请求匹配，则流量量路由到默认后端。

## Ingress 类型

### Single Service Ingress

现在的Kubernetes概念允许公开单个`Service`。也可以通过指定没有规则的默认后端来使用`Ingress`执行此操作。

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80
```



如果使用`kubectl apply -f `创建，能够查看刚刚添加的`Ingress`状态。

```bash
kubectl get ingress test-ingress
```



### Simple fanout

一个fanout配置根据请求的HTTP URL 将流量从单个IP地址路由到多个服务，基于请求的HTTP URL。一个`Ingress`允许将负载均衡器的数量降到最低。例如，

```bash
foo.bar.com -> 178.91.123.132 -> / foo    service1:4200
                                 / bar    service2:8080
```

需要一个这样的`Ingress`例如：

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: service1
          servicePort: 4200
      - path: /bar
        backend:
          serviceName: service2
          servicePort: 8080
```

使用`kubectl apply -f`命令创建`Ingress`:

```bash
kubectl describe ingress simple-fanout-example
```

```bash
Name:             simple-fanout-example
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:4200 (10.8.0.90:4200)
               /bar   service2:8080 (10.8.0.91:8080)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     22s                loadbalancer-controller  default/test
```

只要服务(service1,service2)存在，`Ingress`控制器就会提供满足`Ingress`的特定于实现的负载均衡器。完成后，可以在地址字段查看负载均衡的地址。

> 注意：根据使用的Ingress控制器，可能需要创建default-http-backend服务。

### 基于名称的虚拟主机

基于名称的虚拟主机支持将HTTP流量路由到统一IP地址的多个主机名。

```bash
foo.bar.com --|                 |-> foo.bar.com service1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com service2:80
```

