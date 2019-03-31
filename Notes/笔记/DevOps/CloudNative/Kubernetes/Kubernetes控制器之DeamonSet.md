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