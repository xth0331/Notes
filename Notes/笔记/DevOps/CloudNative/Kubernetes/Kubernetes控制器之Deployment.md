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

