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

`StatefuleSet`中的每个Pod都从`StatefulSet`的名称和Pod的序号中获取其主机名。 构造的主机名模式是`$(statefulset name)-$(ordinal)`。上面的示例将创建三个名为`web-0,web-1,web-2`的Pod。`StatefulSet`可以使用无头服务来控制其Pod的域。此服务管理的域采用以下形式：`$(service name).$(namespace).svc.cluster.local`，其中`cluster.local`是集群域。在创建每个Pod时，它将获得匹配的DNS子域，采用以下形式：`$(podname).$(governing service domain)，其中管理服务由`StatefulSet上的`serviceName`字段定义。

如限制部分所述，您负责创建负责Pod网络身份的Headless服务。

以下是Cluster Domain，Service name， StatefulSet的Pod的DNS名称的一些选择示例。

| Cluster Domain | Service (ns/name) | StatefulSet (ns/name) | StatefulSet Domain              | Pod DNS                                      | Pod Hostname |
| :------------- | :---------------- | :-------------------- | :------------------------------ | :------------------------------------------- | :----------- |
| cluster.local  | default/nginx     | default/web           | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local  | foo/nginx         | foo/web               | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
| kube.local     | foo/nginx         | foo/web               | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

### Stable Storage

Kubernetes为每个VolumeClaimTemplate创建一个PersistentVolume。在上面的示例中，每个Pod将接收一个PersistentVolume，其StorageClass为`my-storage-class`和1Gib的存储。如果未指定StorageClass，则将使用默认的StorageClass。当Pod重新调度到节点上时，其`volumeMounts`将挂载与其PersistentVolume Claims关联的PersistentVolumes。注意，删除Pod或`StatefuleSet`时，不会删除与Pod的PersistentVolume Claims关联的PersistentVolumes。这必须手动完成。

### Pod Name Label

当StatefulSet控制器创建Pod时，它会添加一个标签`statefulset.kubernetes.io/pod-name`，该标签设置为Pod的名称。此标签允许将`Service`附加到`StatefulSet`中特定的Pod。

## Deployment的拓展保证

- 对于具有N个副本的StatefulSet，当部署Pod时，将按顺序从{0…N-1}开始创建他们。
- 当删除Pod时，他们将以{N-1..0}的相反顺序终止。
- 在将缩放操作应用于Pod之前，其之前的必须是Running或Ready。
- 在Pod终止前，所有的后继者必须完全关闭。

### Pod管理策略

在Kubernetes 1.7及更高版本中，StatefulSet允许您放宽其排序保证，同时通过其`.spec.podManagementPolicy`字段保留其唯一性和身份保证。

#### OrderedReady Pod管理

`OrderedReady`pod管理是`StatefulSet`的默认设置。

#### Parallel Pod 管理

`Paraller`pod管理告诉我们`StatefulSet`控制器并行启动或终止所有Pod，并且在启动或终止一个Pod之前不等待pod变成`Ready`或完全终止。

## 更多策略

`OnDelete`更新策略实现了遗留行为。当StatefulSet的`.spec.updateStrategy`设置为`OnDelete`时，不会自动更新StatefulSet中的Pod。用户必须手动删除Pod才能使控制器创建新的Pod，以反映对StatefulSet的`.spec.template`所做的修改。

