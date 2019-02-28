# Kubernetes控制器

## ReplicaSet

ReplicaSet的目的是在任何给定时间维护一组稳定的副本Pod。因此它通常用于保证指定数量的相同Pod的可用性。

### ReplicaSet的工作原理

ReplicaSet是使用字段定义的，包括指定如何识别它可以获取的Pod的选择器，指示它应该维护多少Pod的多个副本，以及一个Pod模板，用于指定它应该创建的新Pod的数据以满足该数量副本标准。然后`ReplicaSet`通过根据需要创建和删除的Pod来达到其目的，以达到所需的数量。当`ReplicaSet`需要创建新Pod时，它使用其Pod模板。

`ReplicaSet`与其Pod的链接是通过Pod的`metadata.ownerReferences`字段，该字段指定当前对象所拥有的资源。`ReplicaSet`获取的所有Pod在其`ownerReferences`字段中拥有的`ReplicaSet`标识信息。通过此链接，`ReplicaSet`知道它维护的Pod的状态并相应地进行计划。

`ReplicaSet`使用其选择器标识要获取的新Pod。如果Pod没有`OwnReference`或者`OwnerReference`不是控制器并且它与`ReplicaSet`的选择器匹配，则它将立即由所述`ReplicaSet`获取。

### 何时使用ReplicaSet

ReplicaSet确保在任何给定时间运行指定数量的Pod副本。但是，`Deployment`是一个更高级别的概念，它管理`ReplicaSet`并为Pod提供声明性更新以及许多其他有用的功能。因此，除非需要自定义更新编排或根本不需要更新，否则建议使用`Deployment`而不是直接使用`ReplicaSet`。这实际上意味着可能永远不需要操作`ReplicaSet`对象：改为使用`Deployment`，并在`spec`部分定义应用程序。

**frontend.yaml:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

将`frontend.yaml`提交到Kubernetes集群将创建已定义的ReplicaSet及其管理的Pod。

```bash
kubectl create -f frontend.yaml
```

```bash
kubectl get rs # 查看创建的ReplicaSet
kubectl describe rc/frontend # 检查ReplicaSet状态
kubectl get Pods # 查看Pods信息
kubectl get pods frontend-xxx -o yaml # 验证这些pod的所有者是否为ReplicaSet

```



### 获取非模板Pod

虽然可以创建裸Pod，但强烈建议确保裸Pod没有与其中任何一个`ReplicaSet`的选择器匹配的标签。因为`ReplicaSet`不限于拥有其模板指定的Pod，他可以按照标签指定的方式获取其他Pod。

```yaml
apiVerison: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  conrainers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---

apiVerison: v1
kind: Pod
metadata: 
  name: pod2
  labels:
    tier: frontend
spec:
  conrainers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
    
```

由于这些Pod没有Controller（或任何对象）作为其所有者引用并与`ReplicaSet`选择器匹配，因此他们被获取。

假设在部署了`ReplicaSet`之后创建了Pod，并设置了其初始Pod副本技术要求：

新的Pod将由`ReplicaSet`获取，然后立即终止，因为`ReplicaSet`将超过其所需计数。

### 编写ReplicaSet

和所有其他KubernetesAPI对象一样，`ReplicaSet`需要`apiVersion`，`kind`和`metadata`字段。对于`ReplicaSet`，`kind`只有`ReplicaSet`。在Kubernetes1.9版本中，`ReplicaSet`类型的`apiVersion`是`apps/v1`默认情况下启用。`apps/v1beta2`已弃用。

#### Pod Template

`.spec.template`是一个Pod模板，也需要`labels`（标签）。例如，`tier: frontend`。注意不要与其他控制器的选择器重叠，一面他们尝试选择这个Pod。

对于`template`的重启策略字段，`。spec.template.spec.restartPolicy`，唯一允许的值是`Always`，这是默认值。

#### Pod Selector

`.spec.selector`字段是标签选择器。如前所述，这些是用于识别可能获得潜在Pod的标签。例如：

```yaml
matchLabels:
	tier: frontend
```

在`ReplicaSet`中，`.spec.template.metadata.labels`必须与`spec.selector`匹配，否则将被API拒绝。

> 对于指定相同`.spec.selector`字段和不同的`.spec.template.metadata.labels`和`.spec.template.spec`字段的两个`ReplicaSet`，每个`ReplicaSet`都会忽略其他`ReplicaSet`创建的Pod。

#### Replicas

可以通过设置`.spec.replicas`来指定应同时运行的Pod数量。`ReplicaSet`将创建/删除其Pod以匹配此数值。

如果没定义`.spec.replicas`，那么默认为1。

### 使用ReplicaSets

#### 删除  ReplicaSet及其Pod

要删除`ReplicaSet`及其所有Pod，使用`kubectl delete`。垃圾收集器默认自动删除所有Pod。

使用REST API 或`client-go`时，必须在`-d`选项中将`propagationPolicy`设置为`Background`或`Foreground`。例如：

```bash
kubectl proxy --port=8000
curl -X DELETE 'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
-d '{"kind": "DeleteOptions","apiVersion"："v1","propagationPolicy":"Foreground"}' \
-H "Content-Type: application/json"
```

#### 仅删除 ReplicaSet

可以使用带有`--cascade=false`选项的`kubectl delete`删除`ReplicaSet`而不影响Pod。

使用REST API 或`client-go`时，必须在`-d`选项中将`propagationPolicy`设置为`Orphan`。例如：

```bash
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/extensions/v1beta1/namespaces/default/replicasets/frontend' \
 -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
 -H "Content-Type: application/json"
```

> 删除原始文件后，可以创建一个新的`ReplicaSet`来替换它。只要就得和新的`.spec.selector`相同，那么将采用旧的Pod。但是，它不会任何使现有的Pod匹配一个新的，不通的模板。要以受控方式将Pod更新为新的规范，使用滚动更新。

#### 从ReplicaSet中删除Pod

可以通过更改标签来从`ReplicaSet`中删除Pod。此技术可用于从服务中删除Pod以进行调试，数据恢复等。以这种方式删除的Pod将自动替换（假设副本数量也未更改）。

#### 伸缩ReplicaSet

只需要更新`.spec.replicas`字段即可轻松扩展或缩小`ReplicaSet`。`ReplicaSet`控制器确保具有匹配标签选择器的所需数量的Pod可用且可操作。

#### ReplicaSet as a Horizontal Pod Autoscaler Target

`ReplicaSet`可以是`HPA`的目标。也就是说，HPA可以自动伸缩`ReplicaSet`。以下是针对我们在上一个实例中创建的`ReplicaSet`的HPA示例：

**hpa-rs.yaml:**

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

将`hpa-rs.yaml`提交给Kubernetes集群，将会创建HPA，HPA根据`ReplicaSet`的Pod的CPU使用情况自动调整目标。

```bash
kubectl create -f hpa-re.yaml
```

另外，可以使用`kubectl autoscale`命令完成相同的操作：

```bash
kubectl autoscale rs frontend --max=10
```

### ReplicaSet的替代品

#### Deployment (推荐)

`Deployment`是一个对象，可以拥有`ReplicaSet`并通过声明式的服务端滚动更新来更新它们及其Pod。虽然`ReplicaSet`可以独立使用，但是主要被`Deployment`用作协调Pod创建，删除，更新的机制。使用`Deployment`时，不必担心管理他们创建的`ReplicaSet`。`Deployment`拥有并管理其`ReplicaSet`。因此，建议在需要`ReplicaSet`时，使用`Deployment`。

#### Bare Pod

与用户直接创建Pod不通，`ReplicaSet`会替换因任何原因而被删除或终止的Pod，例如在节点故障或维护的情况下，例如内核升级。因此，即使应用只需要一个Pod，也建议使用`ReplicaSet`。可以想想它与流程主管类似，只是它监控多个节点上的多个Pod，而不是单个节点上的单个进程。`ReplicaSet`将本地容器重新启动委派给节点上的某个代理程序，例如`Kubelet`或`Docker`。

#### Job

对于预期会自行终止的Pod（批处理作业），使用`Job`而不是`ReplicaSet`。

#### DeamonSet

使用`DaemonSet`代替提供机器级功能的Pod的`ReplicaSet`，例如机器监控或日志记录。这些Pod的生命周期与机器的生命周期有关：Pod需要在其他Pod启动之前在机器上运行，并且当机器准备好重新启动/关闭时可以安全终止。

#### ReplicationController

`ReplicaSet`是`ReplicationController`的后续版本。这两个用途相同，行为相似，只是`ReplicationController`不支持标签用户指南中的基于集合的选择器需求。因此`ReplicaSet`优先于`ReplicationController`。

