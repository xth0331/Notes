# Kubernetes控制器之StatefulSet

`StatefulSet`是用于管理有状态应用程序工作负载的API对象。

管理一组Pod的部署和扩展，并提供有关这些Pod的排序和唯一性保证。

和`Deployment`类似，`StatefulSet`管理基于相同容器规范的Pod。与`Deployment`，`StatefulSet`为其每个Pod维护了一个粘性标识。这些Pod是根据相同的规范创建的，但不可互换：每个Pod都有一个持久的标识符，它在任何重新安排时都会保留。

`StatefulSet`以与任何其他Controller相同的模式运行。在`StatefuleSet`对象中定义所需的状态，`StatefuleSet`控制器进行任何必要的更新已从当前状态到达那里。

## 使用 StatefulSets

`StatefuleSet`对于需要以下一项或多项的应用程序非常有用。

- 稳定，独特的网络标识符。
- 稳定，持久的存储。
- 有序，优雅的部署和扩展。
- 有序的自动滚动更新。

在上文中，稳定与Pod（重新）调度的持久性同义。如果应用程序不需要任何稳定标识符或有序部署，删除或扩展，则应使用提供一组无状态副本的控制器部署应用程序。`Deployment`或`ReplicaSet`可能更适合无状态需求。

## 限制

- `StatefulSet`是1.9版本之前的beta字段，在1.5之前的版本中没有。
- 给Pod的存储必须由`PersistentVolumeProvisioner`根据请求的`StorageClass`进行配置，或者由管理员预先配置。
- 删除和/或缩放`StatefulSet`将不会删除与`StatefuleSet`关联的卷。这样做是为了确保数据安全，这通常比自动清除所有相关的`StatefulSet`资源更有价值。
- `StatefulSet`目前无法要求Headless服务负责Pod的网络身份。需要创建此服务。
- 删除`StatefulSet`时，`StatefulSet`不提供对Pod终止的任何保证。要在`StatefulSet`中实现Pod的有序和正常终止，可以在删除之前将`StatefulSet`缩减为0。

## 组件

下面的实例演示了`StatefulSet`的组件。

- 名为Nginx的Headless服务用于控制网络。
- `StatefulSet`名为web，有一个`Spec`表明nginx容器的3个副本将在唯一的Pod中启动。
- `volumeClaimTemplates`将使用PersistentVolume配置的`PersistentVolumes`提供稳定的存储。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-storage"
      resources:
        requests:
          storage: 1Gi
```

## Pod Selector

必须设置`StatefuleSet`的`.spec.selector`字段以匹配其`.spec.template.metadata.labels`的标签。在1.8版本之前，`.spec.selector`字段在省略时默认为默认值。在之后的版本，如果未指定Pod Selecrot，则会在`StatefulSet`创建期间导致验证错误。

## Pod Identity

`StatefulSet`Pod具有唯一标识，由序数，稳定的网络标识符和稳定的存储组成。无论它在哪个节点上。

### Ordinal Index

对于具有N个副本的`StatefulSet`，`StatefulSet`中的每个Pod将被分配一个整数序数，从0到N-1，在序列中唯一。

### Stable Network ID

`StatefuleSet`中的每个Pod都从`StatefulSet`的名称和Pod的序号中获取其主机名。 