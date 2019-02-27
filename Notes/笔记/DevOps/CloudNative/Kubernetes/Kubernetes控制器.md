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

