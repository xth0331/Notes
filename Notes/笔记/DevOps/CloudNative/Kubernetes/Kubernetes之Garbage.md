# Kubernetes之Garbage Collection

Kubernetes垃圾收集器的作用是删除曾经拥有所有者但不再拥有所有者的某些对象。

## Owners and dependents

一些Kubernetes对象是其他对象的所有者。例如，ReplicaSet是一组Pod的所有者。拥有的对象称为所有者对象的依赖项。每个依赖对象都有一个`metadata.ownerReferences`指向拥有对象的字段。

有时，Kubernetes会自动设置`ownerReference`的值。例如，当你创建一个`ReplicaSet`时，Kubernetes会自动设置`ReplicaSet`中的每个Pod的`ownerReference`字段。在1.8版本中，Kubernetes自动为`ReplicationController`，`ReplicaSet`，`StatefulSet`，`DeamonSet`，`Deployment`，`Job`,`CronJob`创建或采用的对象设置`ownerReference`的值。

还可以通过手动设置`ownerReference`字段来指定所有者和从属者之间的关系。

这是具有三个Pod的ReplicaSet的配置文件：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-repset
spec:
  replicas: 3
  selector: 
    matchLabels:
      pod-is-for: garbage-collection-example
  template:
    metadata:
      labels:
        pod-is-for: garbage-collection-example
    spec:
      containers:
      - name: nginx
        image: nginx
```

如果创建ReplicaSet然后查看Pod元数据，则可以看到`OwnerReferences`字段：

```bash
kubectl get pods -o yaml
```

输出显示Pod所有者的一个名为my-repset的`ReplicaSet`：

```yaml
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: my-repset
    uid: 3f820f9e-5491-11e9-aff9-000c2977af69
```

> 不允许跨namespace的所有者引用，这意味着：
>
> - namespace范围的依赖项只能指定同一namespace中的所有者和集作用域的所有者。
>
> - 集群范围内的依赖只能指定集群范围内的所有者，而不能指定namespace范围的所有者。



