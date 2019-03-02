# Kubernetes控制器之Deployment

`Deployment controller`为Pod和`ReplicaSet`提供声明式更新。

在`Deployment`对象中描述了所需的状态，`Deployment`控制器将实际状态更改为所需状态。您可以定义`Deployment`以创建`ReplicaSet`，或者删除现有的部署并使用新的`Deployment`采用所有资源。

> 不应该管理`Deployment`所拥有的`ReplicaSet`。



## 创建`Deployment`

以下是`Deployment`的示例。它创建了一个`ReplicaSet`来产生三个`nginx`Pod：

**nginx-deployment.yaml：**

```yaml
apiVerison: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        - containerPort: 80
```

在这里例子中：

- 创建名为`nginx-deployment`的`Deployment`，由`.metadata.name`字段指定。

- `Deployment`创建三个副本的Pod，由`replicas`字段指定。

- `selector`字段定义了`Deployment`如何找到要管理的Pod。在这种情况下，只需选择Pod`template`中定义的标签（`app: nginx`）。但是，重要Pod`template`本身满足规则，就可以使用更古扎的选择规则。

  > `matchLabels`是{key, value}的映射。`matchLabels`映射中的单个{key, value}等同于`matchExpressions`的元素，其Key字段，运算符是`in`，value字段，要求是`AND`.

- `template`字段包含以下子字段：

  - 使用`labels`字段将Pod标记为`app: nginx`
  - Pod的`template`规范或`.template.spec`字段表示Pod运行一个nginx容器，容器使用的是Docker Hub的`nginx`镜像，版本是`1.7.9`。
  - 创建一个容器，并使用`name`字段将其命名为`nginx`。
  - 运行版本为`1.7.9`的nginx镜像。
  - 打开80端口，以便容器可以发送和接受流量。

创建上面的`Deployment`使用以下命令：

```bash
kubectl create -f nginx-deployment.yaml
```

运行 `kubectl get deployments`. 输出类似内容：

```bash
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         0         0            0           1s
```

  

检查集群中`Deployment`时，将显示以下字段：

- **NAME** 列出集群中的`Deployment`名称。
- **DESIRED** 显示应用改程序所需的副本数，在创建`Deployment`时定义这些副本。这是理想的状态。
- **CURRENT** 显示当前正在运行的副本数。
- **UP-TO-DATE** 显示已更新以实现所需状态的副本数。
- **AVAILABLE** 显示用户可以使用的应用程序副本数。
- **AGE** 显示应用程序运行的时间。

注意每个字段中的值是如何与`Deployment`中`spec`(规范)中的值相对应：

- 根据`.spec.replicas`字段，所需副本的数量为3。
- 根据`.spec.replicas`字段，当前副本数为0。
- 根据`.spec.updateReplicas`字段，最新副本的数量为0。
- 根据`.status.availableReplicas`字段，可用副本的数量为0。

要查看`Deployment `状态，运行，`kubectl rollout status deployment.v1.apps/nginx-deployment`命令。返回如下

```bash
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment.apps/nginx-deployment successfully rolled out
```

几秒后再次运行`kubectl get deployments`：

```bash
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```

`Deployment`已经创建是三个副本，并且所有副本都是最新的并且可用（Pod状态至少为`Deployment`的`.spec.minReadySeconds`字段的值准备就绪）。

要查看`Deployment`创建的`ReplicaSet`（rs），运行`kubectl get rs`：

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```

注意，副本集的名称始终格式为`[DEPLOYMENT-NAME]-[RANDOM-STRING]`。随机字符串是随机生成的，并使用pod-template-hash。

查看为每个Pod自动生成的标签，运行`kubectl get pods --show labels`，将返回一下内容：

```bash
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
```

 创建的`ReplicaSet`确保始终有三个`nginx`Pod在运行。

#### Pod-template-hash label

> 请勿更改此标签

`Deployment`conrtoller 将`pod-template-hash`标签添加到`Deployment`创建或采用的每个`ReplicaSet`。

此标签可确保`Deployment`的子`ReplicaSet`不重叠。它是通过hasing的`PodTemplate`并使用生成的hash作为添加到`ReplicaSet`选择器的标签值生成的。`PodTemplate`的标签，以及`ReplicaSet`可能 具有任何现有Pod中的标签值生成的。

### 更新 Deployment

> 当且仅当`Deployment`的Pod模板（`.spec.template`）发生改变时，才会触发`Deployment`的部署。例如，如果更新模板的标签或容器镜像。其他更新（例如扩展部署）不会触发部署。



假设现在想要更新Nginx Pod以使用`nginx: 1.9.1`而不是`nginx: 1.7.1`：

```bash
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1
```

或者可以`edit`Deployment，并将`.spec.template.spec.conrtainers[0].image`修改为`nginx: 1.9.1`：

```bash
kubectl edit deployment.v1.apps/nginx-deployment 
```

要查看状态，运行：

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment
```

部署成功后:

```bash
kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s
```

最新副本的数量表示`Deployment`已将副本更新为最新配置。当前副本表示此部署管理的副本总数，可用副本表示可用的当前副本数。

可以运行`kubectl get rs`查看`Deployment`通过创建新的`ReplicaSet`并将其拓展到3个副本来更新Pod以及将旧的`ReplicaSet`缩减为0个副本。

```bash
kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
```

运行`get pods`应该只显示新的Pod：

```bash
kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

下次要更新这些Pod，只需再次更新`Deployment`的Pod模板（`.spec`）。`Deployment`可以确保在更新时只有一定数量的Pod可能关闭。默认情况下，它确保至少比所需的Pod数量少25%（最大不可用25%）。

`Deployment`还可以确保在所需数量的Pod之上只能创建一定数量的Pod。默认情况下，它确保最多比所需的数量多25%。

例如，如果仔细看上面的`Deployment`，会看到它首先创建一个新的Pod，然后删除了一些旧的Pod并创建新的Pod。在有足够数量的新Pod出现之前，它不会杀死旧的Pod，并且在足够数量的旧Pod被杀死之前不会创建新的Pod。它确保可用Pod的数量至少为2，且Pod总数最多为4。

```bash
kubectl describe deployments
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.9.1
    Port:         80/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
  Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
  Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```

在这里，可以看到，当第一次创建`Deployment`时，它创建了一个`ReplicaSet`并直接将其扩展到3个副本。更新`Deployment`时，创建了一个新的`ReplicaSet`并将其扩展为1，然后将就得`ReplicaSet`缩小为2，这样至少有2个Pod可用，最多创建了4个Pod。然后，它继续使用相同的滚动更新策略向上和向下扩展新旧`ReplicaSet`。最后，将在新的`ReplicaSet`中拥有3个可用副本，并将旧的`ReplicaSet`缩减为0。



#### Rollover (aka multiple updates in-flight)