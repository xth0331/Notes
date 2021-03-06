# Kubernetes控制器之DeamonSet

一个`DeamonSet`确保所有（或部分）节点上运行Pod的副本。随着节点添加到集群，将添加Pod。随着节点从集群中删除，这些Pod将被垃圾回收。删除`DeamonSet`将清除它创建的Pod。

`DeamonSet`的一些典型用法：

- 运行集群存储后台守护进程，例如每个节点上的`glusterd`,`ceph`。
- 在每个节点上运行日志收集守护进程，例如`fluentd`或`logstash`。
- 在每个节点上运行节点监控守护进程，例如`Prometheus Node Exporter`，`collectd`，`Dynatrace`，`OneAgent`，`APPDynamics Agent`, `Datadog agent`， `New Relic agetn`， `Ganglia` gmond, 或Instana agent.



在一个简单的例子中，覆盖所有节点的一个`DeamonSet`将用于每种类型的守护进程。更复杂的设置可能会为单一类型的守护进程使用多个`DeamonSet`，但对不同的硬件类型使用不同的标志和/或不同的内存和CPU请求。

## 编写 DaemonSet Spec

### 创建 DaemonSet

可以在`YAML`文件中描述`DeamonSet`，例如，下面`deamonset.yaml`文件描述了一个运行fluentd-elasticsearch Docker镜像的`DeamonSet`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### 必填字段

与所有其他Kubernetes配置一样，`DeamonSet`需要**apiVersion** ,**kind**,和**metadata**字段。

`DeamonSet`还需要一个`.spec`字段。

### Pod Template

`.spec.template`是`.spec`中必填字段之一。

`.sepc.template`是一个Pod模板。它与Pod具有完全相同的架构，除了它是嵌套的并且没有`apiVersion`或`kind`。

除了Pod的必填字段外，`DeamonSet`中的Pod模板还必须指定适当的标签。

`DeamonSet`中的模板必须具有等于Always的`RestartPolicy`或者未指定，默认为Always。

### Pod Selector

`.spec.selector`字段是一个Pod选择器，它的工作方式与Job的`.spec.selector`相同。

必须指定与`.spec.template标签匹配的Pod选择器。`当空时，pod选择器将不在默认，选择器默认与`kubectl apply`不兼容。此外，一旦创建了`DeamonSet`，其`.spec.selector`就无法改变，改变Pod选择器可能导致Pod的无意鼓励，并且发现它对用户来说是混乱的。

`.spec.selector`是由两个字段组成的对象：

- `matchLabels` - 与`ReplicationController`的`.spec.selector`相同。
- `matchExpressions` - 允许通过指定键，值列表以及键和值的相关运算符来构建更复杂的选择器。

指定两者时，结果为AND。

如果指定了`.spec.selector`，则它必须与`.spec.template.metadata.labels`匹配。具有这些不匹配的设置将被API拒绝。

此外，通常不应直接创建任何标签与此选择器匹配的Pod，而异通过另一个DeamonSet或通过其他控制器(例如ReplicaSet)创建。否则，`DeamonSet`控制器会认为这些Pod是由它创建的。Kubernetes不会阻止你这样做。可能希望这样做的一种情况是在节点上手动创建具有不同值的Pod以进行测试。

### 仅在某些节点运行的Pod

如果指定`.spec.template.spec.nodeSelector`,则`DeamonSet`控制器将在该节点选择匹配的节点上创建。同样，如果指定`.spec.template.spec.affinity`，则`DeamonSet`控制器将在与该节点关联相匹配的节点上创建pod，如果为指定任何一个，则`DeamonSet`控制器将在所在节点上创建Pod。

## Deamon Pod如何调度

通常，Pod运行的机器由Kubernetes调度程序选择。但是，由`DeamonSet`控制器创建的Pod已经选择了机器（创建Pod时指定了`.spec.nodeName`，因此调度程序会忽略）。因此：

- `DeamonSet`控制器不遵守节点的不可调度字段
- 即使调度程序尚未启动，`DeamonSet`控制器也可以生成Pod，这可以帮助集群启动。



### 默认调度计划

`DeamonSet`确保所有符合条件的节点都运行Pod副本。通常，Kubernetes导读程序选择Pod运行的节点，但是，`DeamonSet`控件是由`DeamonSet`控制器创建和调度的。这引入以下问题：

- 不一致的Pod行为：等待计划正常Pod以创建并处于`Pending`状态，但`DeamonSet`未在`Pending`状态下创建。让用户感到困惑。
- Pod抢占由默认调度程序处理。启用抢占后，`DeamonSet`控制器将在不考虑Pod优先级和抢占的情况下制定调度决策。

`ScheduleDeamonSetPods`允许使用默认调度程序而不是`DeamonSet`控制器来调度，方法是将`NodeAffinitty`术语添加到`DeamonSet`，而不是`.spec.nodeName`。然后使用默认调度程序将Pod绑定到目标主机。如果`DeamonSet`Pod的节点关联已存在，则替换它。`DeamonSet`控制器仅在创建或修改`DeamonSet`Pod时垂直型这些操作，并且不对`DeamonSet`的`.spec.template`进行任何更改。

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```

此外，`node.kubernetes.io/unschedulable:NoSchedule`容错会自动添加到DeamonSet Pods。在调度`DeamonSet`Pod时，默认调度程序会忽略不可调度的节点。

### 污点和容忍

尽管`DeamonSet`尊重污点和容忍度，但根据相关功能，`DeamonSet` Pods会自动添加以下容忍。

| Toleration Key                           | Effect     | Version | Description                                                  |
| :--------------------------------------- | :--------- | :------ | :----------------------------------------------------------- |
| `node.kubernetes.io/not-ready`           | NoExecute  | 1.13+   | DaemonSet pods will not be evicted when there are node problems such as a network partition. |
| `node.kubernetes.io/unreachable`         | NoExecute  | 1.13+   | 当存在诸如网络分区之类的节点问题时，不会驱逐DeamonSet Pod    |
| `node.kubernetes.io/disk-pressure`       | NoSchedule | 1.8+    |                                                              |
| `node.kubernetes.io/memory-pressure`     | NoSchedule | 1.8+    |                                                              |
| `node.kubernetes.io/unschedulable`       | NoSchedule | 1.12+   | DeamonSet Pod可以通过默认调度程序容忍不可调度的属性。        |
| `node.kubernetes.io/network-unavailable` | NoSchedule | 1.12+   | DaemonSet pods, who uses host network, tolerate network-unavailable attributes by default scheduler. |

## 与Daemon Pods进行通信

在DeamonSet中与Pod通信的一些可能模式是：

- **Push**： DeamonSet中的Pod配置为将更新发送到另一个服务，例如stats数据库。它们没有客户端。
- **NodeIP和已知的端口**：DeamonSet中的Pod可以使用`hostPort`，以便可以通过节点IP访问Pod。客户端以某种方式知道节点IP列表，并按惯例了解端口。
- **DNS**：使用相同的Pod选择器创建headless服务。然后使用`endpoints`资源发现DeamonSet并从DNS检索多个A记录。
- **Service**：使用相同的Pod选择器创建Service，并使用该服务在随机节点上访问守护程序。（无法达到特定节点。）

## 更新 DaemonSet

如果更改了节点标签，DeamonSet会立即将Pod添加到新匹配的节点，并从新匹配的节点中删除Pod。

可以修改DeamonSet创建的Pod，但是，Pod不允许更新所有字段。此外，DeamonSet控制器将在下次创建节点时使用原始模板。

可以删除DeamonSet。如果使用`kubectl`指定`--cascade=false`，则Pod将保留在节点上。然后，可以使用不同的模板创建新的DeamonSet。具有不同模板的新DeamonSet会将所有现有Pod识别为具有匹配标签。尽管Pod模板不匹配，它也不会修改或删除他们。需要通过删除Pod或删除节点来强制创建新的Pod。可在DeamonSet上执行滚动更新。



## DaemonSet的替代品

### Init Scripts

通过在节点上直接启动守护进程（例如使用Init，upstartd或systemd）来运行守护进程当然是可能的。这很好。但是，通过DeamonSet运行此类进程有几个优点：

- 能够像应用程序一样监控和管理守护进程的日志。
- 用于守护进程和应用程序的相同配置语言和工具（例如Pod模板，kubectl）。
- 在具有资源限制的容器中运行守护进程会增加应用程序容器中守护程序之间的隔离。但是这也可以通过在容器中运行守护进程而不是在Pod中运行（例如直接通过Docker启动）来实现。

### Bare Pods

可以直接创建Pod，指定要运行的特定节点。但是，DeamonSet会替换因任何原因而被删除或终止的Pod，例如在节点故障或破坏性节点维护（例如内核升级）的情况的当下。因此，应该使用DeamonSet而不是创建单个Pod。

### Static Pods

可以通过将文件写入`kubelet`监控某个目录来创建Pod。这被称为静态Pod。与DeamonSet按不同，无法使用kubectl或其他API客户端管理静态Pod。静态Pod不依赖apiserver，因此在集群引导情况下非常有用。此外，将来可能不推荐使用静态Pod。

### Deployments

DaemonSets类似于Deployments，因为它们都创建Pod，而那些Pod具有不期望终止的进程（例如Web服务器，存储服务器）。

将部署用于无状态服务（如前端），其中扩展和减少副本数量以及推出更新比控制Pod运行的主机更为重要。当重要的是Pod的副本总是在所有或某些主机上运行，以及何时需要在其他Pod之前启动时，请使用DaemonSet。