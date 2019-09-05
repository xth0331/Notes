# Ingress Controller

为了使`Ingress`资源正常工作，集群必须运行`Ingress Controller`。

与作为`kube-controller-manager`二进制文件一部分运行的其他控制器不同，`Ingress`控制器不会自动与集群一起启动。需要选用适合的控制器进行部署。

Kubernetes作为一个项目目前支持和维护GCE和Nginx控制器。

## 附加控制器

| Ingress Controller                                           |
| ------------------------------------------------------------ |
| [Ambassador](https://www.getambassador.io/)                  |
| [AppsCode Inc.](https://appscode.com/)                       |
| [Contour](https://github.com/heptio/contour)                 |
| Citrix provides                                              |
| F5 Networks provides                                         |
| [Gloo](https://gloo.solo.io/)                                |
| [HAProxy Ingress Controller for Kubernetes](https://github.com/haproxytech/kubernetes-ingress) |
| [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/) |
| [Kong Ingress Controller for Kubernetes](https://github.com/Kong/kubernetes-ingress-controller) |

## 使用多个Ingress控制器

可以在集群中部署任意数量的`Ingress Controller`。创建`Ingress`时，应使用适当的`ingress.class`注释对每个入口，以指定在集群中存在多个Ingress控制器时应使用哪个。

如果没有定义类，则云提供商可能会使用默认的`Ingress`控制器。

理想情况下，所有`Ingress`控制器都应满足此规范，但各种控制器的运行方式略有不通。

