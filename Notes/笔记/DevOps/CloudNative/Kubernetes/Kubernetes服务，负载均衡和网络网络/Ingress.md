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

