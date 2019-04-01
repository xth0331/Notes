# Kubernetes之垃圾回收

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

## 控制垃圾收集器如何删除依赖项

删除对象时，可以指定是否也自动删除对象的依赖项。自动删除依赖项称为级联删除：background和foreground。

如果删除对象而不自动删除其依赖项，则依赖项为孤立对象。

### Foreground cascading deletion

根对象首先进入正在删除状态。在删除过程状态中：

- 通过REST API 仍然可以看到对象
- 设置了对象的`deletionTimestamp`（删除时间戳）
- 对象的`metadata.finalizers`包含值“foregroundDeletion”。

一旦设置了“正在删除”状态，垃圾收集器就会删除对象的依赖项。一旦垃圾收集器删除了所有阻塞依赖项( `ownerReference.blockOwnerDeletion=true`)，他删除了owner对象。



### Background cascading deletion

Kubernetes立即删除所有者对象，然后垃圾收集器在后台删除依赖项。



### 设置级联删除策略

要控制级联删除策略，请在删除对象时在`deleteOptions`参数上设置`propagationPolicy`字段。 可选值包括 “Orphan”, “Foreground”, 或“Background”.

在Kubernetes 1.9之前，许多控制器资源的默认垃圾收集策略是`orphan`。这包括ReplicationController，ReplicaSet，StatefulSet，DaemonSet和Deployment。对于种的`extensions/v1beta1`，`apps/v1beta1`和`apps/v1beta2`组版本，除非另行指定，相关对象默认孤儿。在Kubernetes 1.9中，对于`apps/v1` 组版本中的所有类型，默认情况下会删除依赖对象。



这是一个在后台删除依赖项的示例：

```bash
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
-H "Content-Type: application/json"
```



这是删除前台中的依赖项的示例：

```bash
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
-H "Content-Type: application/json"
```



以下是孤儿依赖的一个例子：

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
-H "Content-Type: application/json"
```

kubectl还支持级联删除。要使用kubectl自动删除依赖项，请设置`--cascade`为true。对于孤儿依赖者，设置`--cascade`为false。默认值为`--cascade` true。

这是一个孤立ReplicaSet的依赖项的例子：

```shell
kubectl delete replicaset my-repset --cascade=false
```























































































[Edit This Page](https://github.com/kubernetes/website/edit/master/content/en/docs/concepts/workloads/controllers/garbage-collection.md)

# Garbage Collection

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.

- [Owners and dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)
- [Controlling how the garbage collector deletes dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#controlling-how-the-garbage-collector-deletes-dependents)
- [Known issues](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#known-issues)
- [What's next](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#what-s-next)

## Owners and dependents

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called *dependents* of the owner object. Every dependent object has a `metadata.ownerReferences` field that points to the owning object.

Sometimes, Kubernetes sets the value of `ownerReference` automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the `ownerReference` field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of `ownerReference` for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.

You can also specify relationships between owners and dependents by manually setting the `ownerReference`field.

Here’s a configuration file for a ReplicaSet that has three Pods:

| [`controllers/replicaset.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/replicaset.yaml) ![Copy controllers/replicaset.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: apps/v1 kind: ReplicaSet metadata:   name: my-repset spec:   replicas: 3   selector:     matchLabels:       pod-is-for: garbage-collection-example   template:     metadata:       labels:         pod-is-for: garbage-collection-example     spec:       containers:       - name: nginx         image: nginx ` |

If you create the ReplicaSet and then view the Pod metadata, you can see OwnerReferences field:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replicaset.yaml
kubectl get pods --output=yaml
```

The output shows that the Pod owner is a ReplicaSet named `my-repset`:

```shell
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

> **Note:** Cross-namespace owner references is disallowed by design. This means: 1) Namespace-scoped dependents can only specify owners in the same namespace, and owners that are cluster-scoped. 2) Cluster-scoped dependents can only specify cluster-scoped owners, but not namespace-scoped owners.

## Controlling how the garbage collector deletes dependents

When you delete an object, you can specify whether the object’s dependents are also deleted automatically. Deleting dependents automatically is called *cascading deletion*. There are two modes of *cascading deletion*: *background* and *foreground*.

If you delete an object without deleting its dependents automatically, the dependents are said to be *orphaned*.

### Foreground cascading deletion

In *foreground cascading deletion*, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:

- The object is still visible via the REST API
- The object’s `deletionTimestamp` is set
- The object’s `metadata.finalizers` contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents (objects with `ownerReference.blockOwnerDeletion=true`), it deletes the owner object.

Note that in the “foregroundDeletion”, only dependents with `ownerReference.blockOwnerDeletion` block the deletion of the owner object. Kubernetes version 1.7 added an [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#ownerreferencespermissionenforcement) that controls user access to set `blockOwnerDeletion` to true based on delete permissions on the owner object, so that unauthorized dependents cannot delay deletion of an owner object.

If an object’s `ownerReferences` field is set by a controller (such as Deployment or ReplicaSet), blockOwnerDeletion is set automatically and you do not need to manually modify this field.

### Background cascading deletion























































































[Edit This Page](https://github.com/kubernetes/website/edit/master/content/en/docs/concepts/workloads/controllers/garbage-collection.md)

# Garbage Collection

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.

- [Owners and dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)
- [Controlling how the garbage collector deletes dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#controlling-how-the-garbage-collector-deletes-dependents)
- [Known issues](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#known-issues)
- [What's next](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#what-s-next)

## Owners and dependents

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called *dependents* of the owner object. Every dependent object has a `metadata.ownerReferences` field that points to the owning object.

Sometimes, Kubernetes sets the value of `ownerReference` automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the `ownerReference` field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of `ownerReference` for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.

You can also specify relationships between owners and dependents by manually setting the `ownerReference`field.

Here’s a configuration file for a ReplicaSet that has three Pods:

| [`controllers/replicaset.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/replicaset.yaml) ![Copy controllers/replicaset.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: apps/v1 kind: ReplicaSet metadata:   name: my-repset spec:   replicas: 3   selector:     matchLabels:       pod-is-for: garbage-collection-example   template:     metadata:       labels:         pod-is-for: garbage-collection-example     spec:       containers:       - name: nginx         image: nginx ` |

If you create the ReplicaSet and then view the Pod metadata, you can see OwnerReferences field:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replicaset.yaml
kubectl get pods --output=yaml
```

The output shows that the Pod owner is a ReplicaSet named `my-repset`:

```shell
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

> **Note:** Cross-namespace owner references is disallowed by design. This means: 1) Namespace-scoped dependents can only specify owners in the same namespace, and owners that are cluster-scoped. 2) Cluster-scoped dependents can only specify cluster-scoped owners, but not namespace-scoped owners.

## Controlling how the garbage collector deletes dependents

When you delete an object, you can specify whether the object’s dependents are also deleted automatically. Deleting dependents automatically is called *cascading deletion*. There are two modes of *cascading deletion*: *background* and *foreground*.

If you delete an object without deleting its dependents automatically, the dependents are said to be *orphaned*.

### Foreground cascading deletion

In *foreground cascading deletion*, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:

- The object is still visible via the REST API
- The object’s `deletionTimestamp` is set
- The object’s `metadata.finalizers` contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents (objects with `ownerReference.blockOwnerDeletion=true`), it deletes the owner object.

Note that in the “foregroundDeletion”, only dependents with `ownerReference.blockOwnerDeletion` block the deletion of the owner object. Kubernetes version 1.7 added an [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#ownerreferencespermissionenforcement) that controls user access to set `blockOwnerDeletion` to true based on delete permissions on the owner object, so that unauthorized dependents cannot delay deletion of an owner object.

If an object’s `ownerReferences` field is set by a controller (such as Deployment or ReplicaSet), blockOwnerDeletion is set automatically and you do not need to manually modify this field.

### Background cascading deletion































































































[Edit This Page](https://github.com/kubernetes/website/edit/master/content/en/docs/concepts/workloads/controllers/garbage-collection.md)

# Garbage Collection

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.

- [Owners and dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)
- [Controlling how the garbage collector deletes dependents](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#controlling-how-the-garbage-collector-deletes-dependents)
- [Known issues](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#known-issues)
- [What's next](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/#what-s-next)

## Owners and dependents

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called *dependents* of the owner object. Every dependent object has a `metadata.ownerReferences` field that points to the owning object.

Sometimes, Kubernetes sets the value of `ownerReference` automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the `ownerReference` field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of `ownerReference` for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.

You can also specify relationships between owners and dependents by manually setting the `ownerReference`field.

Here’s a configuration file for a ReplicaSet that has three Pods:

| [`controllers/replicaset.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/replicaset.yaml) ![Copy controllers/replicaset.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: apps/v1 kind: ReplicaSet metadata:   name: my-repset spec:   replicas: 3   selector:     matchLabels:       pod-is-for: garbage-collection-example   template:     metadata:       labels:         pod-is-for: garbage-collection-example     spec:       containers:       - name: nginx         image: nginx ` |

If you create the ReplicaSet and then view the Pod metadata, you can see OwnerReferences field:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replicaset.yaml
kubectl get pods --output=yaml
```

The output shows that the Pod owner is a ReplicaSet named `my-repset`:

```shell
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

> **Note:** Cross-namespace owner references is disallowed by design. This means: 1) Namespace-scoped dependents can only specify owners in the same namespace, and owners that are cluster-scoped. 2) Cluster-scoped dependents can only specify cluster-scoped owners, but not namespace-scoped owners.

## Controlling how the garbage collector deletes dependents

When you delete an object, you can specify whether the object’s dependents are also deleted automatically. Deleting dependents automatically is called *cascading deletion*. There are two modes of *cascading deletion*: *background* and *foreground*.

If you delete an object without deleting its dependents automatically, the dependents are said to be *orphaned*.

### Foreground cascading deletion

In *foreground cascading deletion*, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:

- The object is still visible via the REST API
- The object’s `deletionTimestamp` is set
- The object’s `metadata.finalizers` contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents (objects with `ownerReference.blockOwnerDeletion=true`), it deletes the owner object.

Note that in the “foregroundDeletion”, only dependents with `ownerReference.blockOwnerDeletion` block the deletion of the owner object. Kubernetes version 1.7 added an [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#ownerreferencespermissionenforcement) that controls user access to set `blockOwnerDeletion` to true based on delete permissions on the owner object, so that unauthorized dependents cannot delay deletion of an owner object.

If an object’s `ownerReferences` field is set by a controller (such as Deployment or ReplicaSet), blockOwnerDeletion is set automatically and you do not need to manually modify this field.

### Background cascading deletion