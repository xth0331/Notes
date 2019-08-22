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

与`pod`和`service`不同，Kubernetes本身并不创建节点，它由云提供商在外部创建，或者存在于您的物理或虚拟机池中。因此，当Kubernetes创建节点时，他会创建一个表示节点的对象。创建后，Kubernetes会检查节点是否有效。例如，如果尝试从以下内容创建节点：

```yaml
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes在内部创建节点对象，并通过基于`metadata.name`字段的运行状况检查来验证节点。如果节点有效，它有资格运行`pod`，否则，对于任何集群活动，将被忽略，知道变为有效。

目前，有三个组件与`kubernetes`节点接口交互：`node controller`,`kubelet`,`kubectl`。



#### Node Controller

`node controller`是Kubernetes Master组件，它管理节点的各个方面。

`node controller`在节点的生命周期中具有多个角色。第一种是在注册是为节点分配CIDR（如果打开的CIDR分配）。

第二个是使`node controller`的内部节点列表与云提供商的可用计算机列表保持同步。在云环境中运行时，只要节点运行状况不佳，`node controller`就会寻味云提供商该节点的VM是否可用。如果不可用，则`node controller`从其节点列表中删除该节点。

第三种是监控节点的健康状况。

#### 节点自注册



当kubelet标志`--register-node`为true（默认值）时，kubelet将尝试向API服务器注册自己。这是大多数发行版使用的首选模式。

对于自行注册，可以使用以下选项启动kubelet：

- `--kubeconfig` - 凭证路径，以向apiserver验证自身。
- `--cloud-provider` - 如何与云提供商交谈以阅读有关自身的元数据。
- `--register-node` - 自动注册API服务器。
- `--register-with-taints`- 使用给定的taints列表注册节点（以逗号分隔`<key>=<value>:<effect>`）。No-op如果`register-node`是假的。
- `--node-ip` - 节点的IP地址。
- `--node-labels`- 在群集中注册节点时添加的标签。
- `--node-status-update-frequency` - 指定kubelet将节点状态发布到master的频率。

#### 手动节点管理

集群管理员可以创建和修改节点对象。

如果管理员希望手动创建节点对象，请设置kubelet标志 `--register-node=false`。

管理员可以修改节点资源（无论设置如何`--register-node`）。修改包括在节点上设置标签并将其标记为不可调度。

节点上的标签可以与pod上的节点选择器结合使用以控制调度，例如，将pod限制为仅有资格在节点的子集上运行。

将节点标记为不可调度可防止将新pod调度到该节点，但不会影响节点上的任何现有pod。这在节点重启等之前作为准备步骤很有用。例如，要标记节点不可调度，请运行以下命令：

```shell
kubectl cordon $NODENAME
```

> **注意：**由DaemonSet控制器创建的Pod绕过Kubernetes调度程序，不遵守节点上的不可调度属性。这假设守护进程属于机器，即使它在准备重新启动时正在耗尽应用程序。

#### 节点容量

节点的容量（cpus的数量和内存量）是节点对象的一部分。通常，节点在创建节点对象时注册自己并报告其容量。如果您正在进行手动节点管理，则需要在添加节点时设置节点容量。

Kubernetes调度程序确保节点上的所有pod都有足够的资源。它检查节点上容器请求的总和不大于节点容量。它包括由kubelet启动的所有容器，但不包括由容器运行时直接启动的容器，也不包括在容器外部运行的任何进程。



## API对象

Node是Kubernetes REST API中的顶级资源。