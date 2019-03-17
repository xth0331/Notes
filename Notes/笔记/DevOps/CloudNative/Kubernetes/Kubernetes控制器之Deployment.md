# Kubernetes控制器之Deployment

`Deployment controller`为Pod和`ReplicaSet`提供声明式更新。

在`Deployment`对象中描述了所需的状态，`Deployment`控制器将实际状态更改为所需状态。您可以定义`Deployment`以创建`ReplicaSet`，或者删除现有的部署并使用新的`Deployment`采用所有资源。

> 不应该管理`Deployment`所拥有的`ReplicaSet`。



### 创建`Deployment`

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



#### Rollover

每次`Deployment`控制器观察到新的`Deployment`对象时，如果没有现有的`ReplicaSet`，则会创建`ReplicaSet`以显示所需的Pod。现有的`ReplicaSet`控制其标签与`.spec.selector`匹配但其模板与`.spec.template`不匹配的Pod缩小。最终，新的`ReplicaSet`将缩放为`.spec.replicas`将缩放为`.spec.replicas`，并且所有的`ReplicaSet`将缩放为0。

如果对现有的`Deployment`进行更新，`Deployment`将根据更新创建一个新的`ReplicaSet`并开始向上拓展，并将饭庄之前正在扩展的`ReplicaSet` - 它将把它添加到旧的`ReplicaSet`列表中并开始缩小它。

例如，假设创建了一个`Deployment`创建5个`nginx: 1.7.9`的副本，但随后更新`Deployment`以创建`nginx: 1.9.1`的五个副本，此时仅创建了3个`nginx: 1.7.9`的副本。在这种情况下，`Deployment`将立即开始杀死它创建的3个`nginx: 1.7.9`Pod，并将开始创建`nginx: 1.9.1`Pod。在更改之前，不会等该`nginx: 1.7.9`的副本。

#### Label selector 更新

通常不鼓励进行标签选择器更新，建议事先规划`selector`，在任何情况下，如果需要执行标签选择器更新，务必小心谨慎，确保以掌握含义。

> 在API版本的`app/v1`中，`Deployment`的标签选择器创建后是不可变的。

- 选择器添加要求使用新标签更新`Deployment`规范中的Pod模板标签，否则将返回验证错误。此更改是非重叠的，这意味着新选择器不会选择使用旧选择器创建的`ReplicaSet`和`Pod`，从而导致孤立所有旧`ReplicaSet`并创建新的`ReplicaSet`。
- 选择器更新 - 也就是说，更改选择建中的现有值 - 导致与添加相同的行为。
- 选择器移除 - 也就是说，从部署选择器中删除现有`key` - 不需要对Pod模板标签进行任何更改。没有现有的`ReplicaSet`是孤立的，并且未创建新的`ReplicaSet`，但请注意，已删除的标签仍然存在于任何现有Pod中和`ReplicaSet`中。



### 回滚 Deployment

有时可能要回滚`Deployment`；例如，当部署不稳定时，例如，崩溃循环。默认情况下，所有`Deployment`的历史记录都保留在系统中，以便可以随时回滚。

> 触发Deployment的部署时会创建Deployment的修订版。这意味着当且仅当`Deployment`部署的Pod模板(`.spec.template`)发生更改时才会创建新修订，例如，如果更新模板的标签或容器镜像。其他更新（例如扩展部署）不会创建部署版本，因此可以方便地同时进行手动或自动拓展。这意味着当回滚到早期版本时，仅回滚`Deployment`的Pod模板部分。

假设在更新`Deployment`时输入了拼写错误，方法是将镜像名称`nginx: 1.91`替换为`nginx: 1.9.1`：

```bash
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1  --record=true
```

将看到旧的`replicas`数量是2，新的`replicas`数量是1.

```bash
kubectl get rs
```

查看创建的Pod，可以看到由新`ReplicaSet`创建的Pod状态是`ImagePullBackoff`。

> 注意：`Deployment`控制器将自动停止错误的，并将停止扩展新的`ReplicaSet`。这取决于指定的`rollingUpdate`参数（特别是`maxUnavailable`）。默认情况下，Kubernetes将设置为25%。

```bash
kubectl describe deployment
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.91
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
```

要解决此问题，需要回滚到稳定的以前版本的`Deployment`。

#### 检查Deployment的历史记录

首先检查此部署的修订：

```bash
kubectl rollout history deployment nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.91 --record=true
```

`CHANGE-CAUSE`从`Deployment`注释`kubernetes.io/change-cause`复制到创建时的修订版本。可以通过以下方式指定`CHANGE-CAUSE`消息：

- 使用`kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to update to 1.9.1"`注释`Deployment`。
- 附加`--record`标志以保存正在更改资源的`kubectl`命令。
- 手动编辑资源清单。

要进一步查看每个修订的详细信息，运行：

```bash
kubectl rollout history deployment.v1.apps/nginx-deployment --reversion=2
```

#### 回滚到以前的版本

现在，已决定撤销当前并回滚到上一个版本：

```bash
kubectl rollout undo deployment nginx-deployment 
```

或者，通过`--to-revision`指定回滚到特定修订：

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=2
```

`Deployment`现在回滚到以前的稳定版本。可以看到从`Deployment`控制器生成用于回滚到版本2的`DeploymentRollback`事件。

```bash
kubectl deployment nginx-deployment
kubectl describe deployment nginx-deployment
```

### 扩展Deployment

可以使用以下命令扩展部署：

```bash
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```

假设集群中开启了HPA（Horizontal Pod Autoscaling），可以为`Deployment`设置`autoscaler`，根据现有Pod的CPU利用率选择要运行的最小和最大Pod数量。

```bash
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

#### 比例缩放

`RollingUpdate Deployment`支持同时运行多个版本的应用程序。当你或自动扩展器扩展（正在进行或暂停）的`RollingUpdate Deployment`时，`Deployment`控制器将平衡现有活动`ReplicaSet`中其他副本（ReplicaSet with Pods）以降低风险。这称为比例缩放。例如，正在运行具有10个副本的`Deployment`，`maxSurge=3`，`maxUnavailable=2`。

```bash
kubectl get deploy
```

更新到一个新的镜像，该镜像恰好在集群中无法解析。

```bash
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
```

镜像更新使用`ReplicaSet-deployment-1989198191`开始新的部署，但是由于上面提到的`maxUnavailabel`要求被阻止。

```bash
kubetl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```

然后在出现一个新的`Deployment`扩展请求。`autoscaler`(自动缩放器)将`Deployment`的副本增加到15。`Deployment`控制器需要决定在哪里添加新的5个副本。如果没有使用比例缩放（proportional scaling），则五个副本都将添加到新的`ReplicaSet`中。通过比例缩放，可以在所有ReplicaSet上传播其他副本。具有最多副本的`ReplicaSet`和较低比例的较大比例转换到具有较少副本的ReplicaSet。任何剩余都会添加到具有最多副本的`ReplicaSet`中。零副本的`ReplicaSet`不会按比例放大。

在上面的示例中，将3个副本添加到旧的`ReplicaSet`，并将2个副本添加到新的`ReplicaSet`。假设新副本变得健康，推出过程最终应将所有副本添加到新的`ReplicaSet`。

```bash
kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

### 暂停和恢复Deployment

你可以在出发一个或多个更新之前暂停`Deployment`，然后恢复它。这将允许你在暂停和恢复之间应用多个修复，而不会出发不必要的部署。

例如，使用刚刚创建的`Deployment`:



```bash
kubectl get deploy
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     3         3         3            3           1m
kubectl get rs
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         1m
```

通过运行以下命令暂停：

```bash
kubectl rollout pause deployment.v1.apps/nginx-deployment
```

然后更新部署的镜像：

```bash
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1
```

注意，没有新的，

可以根据需要进行更多更新，例如，更新使用的资源：

```bash
kubectl set resourcess deployment.v1.apps/nginx-deployment -c=nginx --limit=cpu=200m,memory=512Mi
```

暂停之前`Deployment`的初始状态将继续其功能，但只要`Deployment`要部署暂停，`Deployment`的新更新将不会产生任何影响。

最后，恢复`Deployment`并观察一个新的`Deployment`，提供所有新的数据。

```bash
kubectl rollout resume deployment.v1.apps/nginx-deployment
kubectl get rs -w
kubectl get rs
```

### Deployment 状态

部署在其生命周期中进入各种状态。它可以退出新的`ReplicaSet`时进行，可以完成，也可以无法进行。

#### Progressing Deployment

当执行以下任务之一时，Kubernetes将`Deployment`标记为进度：

- `Deployment`创建了一个新的`ReplicaSet`。
- `Deployment`正在扩展其最新的 `ReplicaSet`。
- `Deployment`正在缩减其旧的`ReplicaSet`。
- 新Pod已准备就绪或可用（至少准备MinReadySeconds）。

可以使用`kubectl rollout status`监视`Deployment`的进度。

#### 完成 Deployment

Kubernetes在具有以下特征时将`Deployment`标记为完成：

- 与`Deployment`关联的所有副本都以更新为指定的最新版本，这意味着已请求的更新完成。
- 可以使用与`Deployment`关联的所有副本。
- 没有旧的`Deployment`副本正在运行。

可以使用`kubectl rollout status`检查`Deployment`是否已完成。如果成功完成，将返回零退出代码：

```bash
kubectl rollout status deployment.v1.apps/nginx-deployment
echo $?  # if the rollout completed successful，return 0
```

#### Deployment 失败

`Deployment`可能会在尝试部署最新的`ReplicaSet`时遇到困难，无法完成`Deployment`，这可能由于以下一些因素造成：

- 配额不足
- `Readiness`探针失败
- 镜像拉取错误
- 权限不足
- 限制范围
- 应用程序运行时配置错误

检测此情况的一种方法是在`Deployment`规范中指定解释时间参数:(`.spec.progressDeadlineSeconds`)。`.spec.progress.DeadlineSeconds`表示`Deployment`控制器在指示（在`Deployment`状态中）`Deployment`进度已停止之前的等待秒数。

以下`kubectl`命令使用`progressDeadlineSeconds`设置规范，以使控制器报告在10分钟后缺少部署进度：

```bash
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

超过截止时间后，`Deployment`控制器会将`DeploymentCondition`与以下属性一起添加到`Deployment`的`.status.conditions`：

- Type=Progressing
- State=False
- Reason=ProgressDeadlineExceeded

由于设置的超时时间较短或者由于任何其他可被视为瞬态的错误，可能会遇到`Deployment`的暂时性错误。例如，假设您的配额不足。如果描述`Deployment`，将注意到以下部分:

```bash
kubectl describe deployment nginx-deployment
```

```bash
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

最终，一旦超出部署进度截止日期，Kubernetes将更新状态和进度条件的原因：

```bash
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

可以通过缩小部署，缩小可能正在运行的其他控制器或增加命名空间中的配额来解决配额不足的问题。如果您满足配额条件，然后部署控制器完成“部署”卷展栏，您将看到部署状态更新成功条件（`Status=True`和`Reason=NewReplicaSetAvailable`）。



### 清理策略

可以在`Deployment`中设置`.spec.revisionHistoryLimit`字段，以指定要保留此`Deployment`的旧`ReplicaSet`数量。其余的将在后台进行垃圾回收，默认情况下，是10。

> 此字段设置为０将导致清理所有的历史记录，从而导致　`Deployment`无法回滚。

### 用例



#### 金丝雀部署

如果要使用`Deployment`将发布部署到用户或服务器的子集，则可以按照惯例资源中描述的`canary`模式创建多个部署，每个版本一个。



### 编写`Deployment`规范（spec）

与其他所有Kubernetes配置一样，`Deployment`需要`apiVersion`，`kind`和`metadata`字段。还需要`.spec`字段。

#### Pod Template

`spec.template`是`.spec`中唯一必须的字段。

`.spec.template`是一个Pod模板。它与Pod具有完全相同的架构，除了它是嵌套的并且没有`apiVerison`和`kind`。

除了Pod的必填字段外，`Deployment`中的Pod模板还必须指定适当的标签和适当的重启策略。对于标签，确保不要与其他控制器重叠。

#### Replicas

`.spec.replicas`是一个可选字段，用于指定所需的Pod数量，默认为1。

#### Selector

`.spec.selector`是一个可选字段，它指定`Deployment`所针对的Pod的标签选择器。

`.spec.selector`必须与`.spec.template.metadata.labels`匹配，否则它将被API拒绝。

在API版本中的`apps/v1`,`.spce.selector`和`.metadata.labels`中，如果未设置，则不会默认为`.spec.template.metadata.labels`。所以必须明确设置。注意，在`apps/v1`中创建`Deployment`后，`.spec.selector`是不可变的。

如果其模板与`.spec.template`不同，或此类Pod的总数超过`.spec.replicas`，则`Deployment`可以终止其标签与选择器匹配的Pod。如果Pod的数量小于所需的数量，它会带有`.spec.template`的新Pod。

> 不应通过创建另一个`Deployment`，或通过创建另一个控制器（如`ReplicaSet`或`ReplicationController`）来创建其标签与此选择器匹配的其他Pod。如果这样做，第一个`Deployment`认为它创建了其他这些pod。Kubernetes并没有阻止你这样做。

如果有多个具有重叠选择器的控制器，控制器将互相争斗并且行为不正确。

#### Strategy

`.spec.strategy`指定用于替换旧Pod的策略。`.spec.strategy.type`可以是`RollingUpdate`或`Recreate`。`RollingUpdate`是默认值。

##### Recreate Deployment









