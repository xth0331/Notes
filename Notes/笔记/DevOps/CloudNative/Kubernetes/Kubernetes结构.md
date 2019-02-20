# Kubernetes结构

## Node

Node是Kubernetes中的工作机器，以前成为`minion`,node可以是VM或物理机，取决于集群。每个node都包含运行`pod`所需的服务，并由`master`管理。node上的服务包括容器运行时，`kubelet`和`kube-proxy`。

### Node状态

Node的状态包含以下信息：

- `addresses`

  这些字段的使用取决于云提供商或裸机配置。

  - `HostName`: node的内核报告的主机名。可以通过`kubectl --hostname-override`参数覆盖。
  - `ExternalIP`：通常是可从外部路由的节点的IP地址（可从集群外部获得）。
  - `InternalIP`：通常是尽在集群内可路由的节点的IP地址。

- `condition`

  conditions字段描述了所欲正在运行的节点状态。

  | Node Condition         | Description                                                  |
  | ---------------------- | ------------------------------------------------------------ |
  | **OutOfDisk**          | `True`,如果节点上的可用空间不足以添加新的`pod`,否则`False`   |
  | **Ready**              | `True`,如果节点健康并准备好接受`pod`<br />`False`，如果节点不健康且不接受`pod`<br />`Unknown`，如果节点控制器没有从最后一个节点接收到`node-monitor-grace-period` |
  | **MemoryPressure**     | `True`,如果节点内存上存在压力，<br />`False`，节点内存没有压力。 |
  | **PIDPressure**        | `True`,如果节点进程上存在压力，<br />`False`，节点进程没有压力。 |
  | **DiskPressure**       | `True`,如果节点磁盘空间上存在压力，<br />`False`，节点磁盘空间上没有压力。 |
  | **NetWorkUnavailable** | `True`,如果未正确配置节点的网络，<br />否则为`False`         |

  节点条件表示为`JSON`对象，例如，以下描述了健康节点。

  ```json
  "conditions": [
    {
       "type": "Ready",
        "status": "True"
    }
  ]
  ```

  如果就绪的状态保持位置或错误的时间超过`pod-eviction-timeout`，则会将参数传递给`kube-controller-manager`，并且节点控制器会调节该节点上的所有Pod以供删除。默认住处超时持续时间为五分钟。在某些情况下，当节点无法访问时，apiserver无法用于节点上的kubelet通信。在重新简历与apiserver的通信之前，不能将删除pod的决定传给kubelet。同时，计划删除的pod可以继续在分区节点上运行。

  在1.5之前的Kubernetes版本中，节点控制器会强制删除这些无法访问的pod。但是，在1.5及更高版本中，节点控制器不会强制删除容器，直到确认它们已停止在群集中运行。您可以看到可能在无法访问的节点上运行的pod处于`Terminating`或`Unknown`处于状态。如果节点永久离开群集，如果Kubernetes无法从底层基础架构推断出，则群集管理员可能需要手动删除节点对象。从Kubernetes中删除节点对象会导致节点上运行的所有Pod对象从apiserver中删除，并释放它们。

  在版本1.12中，`TaintNodesByCondition`功能被提升为beta版，因此节点生命周期控制器会自动创建表示条件的污点。 类似地，调度程序在考虑节点时忽略条件; 相反，它会查看Node的污点和Pod的容忍度。

  现在，用户可以在旧调度模型和更灵活的新调度模型之间进行选择。

- `capacity`

  描述节点上可用的资源：CPU，内存以及可以在节点上调度的最大pod数量。

- `info`

  有关节点的一般信息，例如内核版本Kubernetes版本，container run-time版本，操作系统名称等。信息 由`kubelet`从各节点收集。



### 管理