# Kubernetes之Pod

## Pod概览

### 认识Pods

`pod`是Kubernetes的基本构建单元——是创建或部署Kubernetes对象模型中最小最简单的单元。一个`pod`表示集群上正在运行的进程。

一个`pod`包含一个应用容器（或者，在某些情况下，是多个容器）， 存储资源，一个独立网络IP，以及控制容器运行方式的选项。一个`pod`表示`deployment`单元：Kubernetes中的单个应用程序实例，可能由单个容器或少量紧耦合并共享资源的容器组成。

Docker是Kubernetes Pod中最常用的容器运行时，但Pod也支持其他容器运行时。

Kubernetes及群众Pod可以以两种主要方式使用：

- **运行单个容器Pod**

  *one-container-per-Pod*模型是最常见的；在这种情况下，可以将Pod视为单个容器的包装，而Kubernetes直接管理Pod而不是容器本身。

- **运行多个需要协同工作的容器的Pod**

  Pod可以封装由多个协同工作的容器组成的应用程序，这些容器紧密耦合并且需要共享资源。这些共处一地的容器可能形成一个统一的服务单元-一个容器从共享卷想公众提供文件，而一个单独的`sidecar`容器刷新或更新这些文件。Pod将这些容器和存储资源作为单个可管理实体包装在一起。

  

每个Pod都用于运行给定应用程序的单个实例。如果要水平拓展应用程序（例如，运行多个实例），则应使用多个Pod，每个实例一个。在Kubernetes中，这通常被称为*replication*。复制Pod通常由称为`controller`的抽象创建和管理。

#### Pod如何管理多个容器

Pod旨在支持多个写作流程（作为容器），形成一个有凝聚力的服务单元。Pod中的容器自动位于及群众的同一物理或虚拟机上，并共同调度。容器可以共享资源和依赖关系，彼此通信，并协调它们何时以及如何终止。

 请注意，在单个Pod中对过个共存和共同管理的容器进行分组时一个相对高级的用例。直营在容器紧密耦合的特定情况下使用此模式。例如，可能有一个容器充当共享卷中文件的web服务器，以及一个单独的`sidecar`容器，用于远程源更新这些文件。如图所示：

![Pod](https://blog-image.nos-eastchina1.126.net/pod1.jpg)



Pod为期组成容器提供两种共享资源：网络和存储。

- 网络

  每个Pod都分配唯一的IP地址。Pod中的每个容器都共享网络namespace，包括IP地址和网络端口。Pod中的容器可以使用`localhost`相互通信。当Pod中的容器与Pod外部的实体通信时，必须协调它们图和使用共享网络资源（例如端口）。

- 存储

  Pod可以指定一组共享存储卷。Pod中的所有容器都可以访问共享卷，允许这些容器共享数据。如果需要重新启动其中一个容器，则卷还允许Pod中的持久数据存活。



### 使用Pod

很少直接在Kubernetes创建单独的Pod。这是因为Pod被设计为相对短暂的一次性实体。当创建Pod（直接创建或由Controller间接创建）时，它将被安排在集群中的节点上运行。Pod保留在该节点上，知道进程终止，Pod对象将被删除，Pod因资源不足而被驱逐，或者Node失败。

> 不应重启Pod，Pod中的容器。Pod本身不会运行，但是容器运行的环境会持续存在，知道删除为止。

Pod本身不能自我修复。如果将Pod调度到失败的节点，则删除Pod；同样，由于缺乏资源或节点维护，Pod无法在驱逐中存活。Kubernetes使用更高级别的抽象，称为`Controller`，它处理管理相对可处理的Pod实例的工作。因此，虽然可以直接使用Pod，但在Kubernetes中使用Controller管理Pod更为常见。

#### Pods and Controllers

`Controller`可创建和管理对个Pod，处理复制和部署，并在集群范围内提供自修复功能。例如，如果节点发生故障，`Controller`可能会通过在不同节点上安排相同替代品来自动替换Pod。

包含一个或多个Pod的控制器包括：

- Department
- StatefulSet
- DeamonSet

通常，控制器使用提供的Pod模板来创建它负责的Pod。

### Pod Templates

Pod模板是Pod的规范，包含在其他对象中，例如`Replication Controllers`,`Jobs`，`DeamonSets`。控制器使用Pod模板制作实际的Pod。下面的示例是Pod的简单清单，其中包含一个打印消息的容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c','echo Hello Kubernetes! && 3600']
```

Pod模板不是指定所有副本的当前所需状态,而是像饼干模具，切割饼干后，饼干与模具无关。

对模板的后续更改甚至切换到新模板对已创建的Pod没有直接影响。类似地，随后可以直接更新由`replication controller`创建的Pod。这与Pod有意对比，Pod确实指定了书序Pod的所有容器的当前所需状态。这种方法从根本上简化了系统语义并增加了原语的灵活性。

## Pod

Pod是可以创建和管理的Kubernetes计算的最小可部署单元。

### 什么是Pod

一个Pod（在一群鲸鱼或豆荚中 ）是一组一个或多个容器，具有共享存储/网络，以及如何运行容器的规范。pod的内容始终位于同一位置并共同调度，并在共享上下文中运行。pod模拟特定于应用程序的逻辑主机。包含一个或多个相对紧密耦合的应用程序容器 - 在预容器世界中，在同一物理或虚拟机上意味着在同一逻辑主机上运行。

虽然Kubernetes支持的容器运行时多于Docker，但Docker是最常见的。

Pod的共享上下文是一组Linux Namespaces，cgroup。以及可能的隔离方面 - 与隔离Docker容器相同的东西。在Pod的上下文中，各个应用程序可能会应用进一步的子隔离。

Pod中的容器共享IP地址和端口空间，并且可以通过他们找到彼此。他们还可以使用标准的进程间通信（如SystemV信号量或POSIX共享内存）相互通信。不同pod中的容器具有不同的IP地址。并且在没有特殊配置的情况下无法通过IPC通信。这些容器通常通过Pod IP地址相互通信。

Pod中的应用程序还可以访问共享卷，共享卷被定义为Pod的一部分，可以挂在到每个应用程序的文件系统中。

就Docker构造而言，Pod被建模为一组具有共享命名空间和共享卷的Docker容器。

与单个应用程序容器一样，Pod被认为是相对短暂的（不是持久）实体。正如Pod的生命周期中所讨论的，创建Pod，分配唯一ID（UID），并调度到它们保留的节点，直到终止（根据重启策略）或删除。如果节点终止，则在超时期限后，将调度计划到该节点的Pod进行删除。给定的Pod（UID定义）不会“重新安排”到新节点；相反，它可以被相同的Pod替换，如果需要，甚至可以使用相同的名称，但是使用新的UID。

当某些东西被认为具有与容量相同的生命周期时，例如卷，这意味着只要该容器（具有该UID）存在就存在。如果由于任何原因删除了Pod，计时创建了相同的替换，相关的东西（例如卷）也会被销毁重新创建。

### Pod的动机

#### 管理

Pod是多个合作过程模式的模型，形成了一个有凝聚力的服务单元。它们通过提供比其组成应用程序更高级别的抽象来简化应用程序的部署和管理。Pod用作部署，水平扩展和复制的单元。对容器中的容器自动处理共置（共同调度），共享命运（例如终止），协调复制，资源共享和依赖关系管理。

#### 资源共享和通信

Pod可以实现其成员之间的数据共享和通信。

Pod中的应用程序使用相同的网络命名空间（相同的Ip和端口空间），因此可以互相找到平使用它们进行通信。因此，Pod中的应用程序必须协调它们对端口的使用。每个Pod在平面共享网络空间中具IP地址，该网络空间与网络上的其他物理计算机和Pod完全通信。

除了定义在Pod中运行的应用程序容器之外，Pod还指定了一组共享存储卷。卷使数据能够在容器重新启动后继续存在，并在容器内的应用程序之间共享。



### 豆荚的使用

 Pod可用于托管垂直集成的应用程序堆栈（例如LAMP）。但其主要动机是支持协同定位，共同管理的帮助程序，例如：

-  内容管理系统，文件和数据加载器，本地缓存管理器。
- 日志和检查点备份，压缩，旋转，快照等。
- 数据变更观察者，日志和监控适配器，活动发布者等。
- 代理，网桥和适配器。
- 控制器，管理器，配置器和更新器。

通常，单个Pod不用于运行同一个应用程序的多个实例。

### 考虑的替代方案

为什么不在一个容器中运行多个程序？

1. 透明度，是基础架构内的容器对基础架构可见，使基础架构能够为这些容器提供服务，例如进程管理和资源监控。这为用户提供了许多便利。
2. 解耦软件依赖关系。各个容器可以独立地进行版本化，重建和重新部署。Kubernetes有一天可能会支持单个容器的实时更新。
3. 便于使用，用户无需运行自己的流程管理器，担心信号和退出代码的传播。
4. 效率，由于基础设施承担更多责任，因此容器可以更轻量。

为什么不支持基于亲和力的容器协同调度，这种方法可以提供协同定位，但不会提供pod的大部分好处，例如资源共享，IPC保证共享和简化管理。

### 豆荚的耐久性

Pod不应被视为长久实体。它们将无法在调度故障，节点故障或其他驱逐（例如缺乏资源）或节点维护的情况下存活。

通常，用户不需要直接创建Pod。他们几乎总是使用控制器，例如，`Deployments`控制器提供集群范围内的自我修复，以及复制和部署管理。像`StatefulSet`这样的控制器也可以为有状态的Pod提供支持。使用API作为主要面向用户的原语在集群调度系统中相对常见。

Pod以便于：

- 调度程序和控制器可插拔性
- 支持Pod级别操作，无需通过控制器API代理他们
- Pod生命周期与控制器生命周期的解耦
- 控制器和服务的分离
- 具有集群级功能的`Kubelet`级功能的清晰组合 - Kubelet实际上是Pod控制器
- 高可用性，它们将期望在终止之前更换Pod，并且肯定在删除之前，例如在激活驱逐或图像预取的情况下。

### 终止Pod

因为Pod表示集群中节点上正在运行的进程，所以允许这些进程不在需要时优雅地终止（与使用KILL信号杀死进程并且没有清理）非常重要。用户应该能够请求删除并指导进程何时终止，但也能够确保删除最终完成。当用户请求删除Pod时，系统会在允许Pod强制终止之前记录预期的宽期限，并将TERM信号发送到每个容器中的主进程。宽限期到期后，KILL信号将发送到这些进程，然后从API服务器中删除该Pod。如果在等待进程终止时重新启动Kubelet或容器管理器。

一个示例流程：

1. 用户发送删除Pod的命令，默认宽限期（30秒）
2. API服务器中的Pod随着时间推移而更新，其中Pod被视为“死亡”以及宽限期。
3. 在客户端命令中列出时，Pod显示为`Terminating `
4. 当`kubelet`看到Pod已经被标记为`Terminating`，开始了Pod关闭过程（跟步骤3同时）
   1. 如果其中一个Pod的容器定义了一个`preStop`钩子，则会在容器内调用它。如果`preStop`宽限期到期后钩子仍在运行，则以延迟宽限期调用
   2. 容器被发送TERM信号。注意，并非Pod中的所有容器都会同时收到TERM信号，并且`preStop`如果它们关闭顺序很重要，则每个容器都需要一个钩子。
5. Pod从断点列表中删除以进行维护，并且不再被视为复制控制器的运行Pod集的一部分。缓慢关闭的Pod无法继续提供流量最为负载均衡从中删除它们。
6. 当宽限期到期时，仍然在Pod中运行的任何进程都将被SIGKILL杀死。
7. kubelet将通过设置宽期限0（立即删除）完成删除API服务器上的Pod。Pod从API中消失，客户端不再可见。



#### 强制删除pod

强制删除pod被定义为立即从群集状态和etcd删除pod。当执行强制删除时，许可证持有者不会等待来自kubelet的确认该pod已在其运行的节点上终止。它会立即删除API中的pod，以便可以使用相同的名称创建新的pod。在节点上，设置为立即终止的pod在被强制终止之前仍将被给予一个小的宽限期。

强制删除可能对某些Pod有潜在危险，应谨慎执行。

### pod容器的特权模式

Pod中的任何容器都可以使用容器规范中的`SecurityContext`上的特权标记启用`privileged`模式。 这对于想要使用Linux功能（如操作网络堆栈和访问设备）的容器非常有用。容器内的进程获得与容器外部进程可用的几乎相同的权限。使用特权模式，将网络和卷插件编写为不需要编译到`kubelet`的独立pod应该更为容易。

## Pod生命周期

### Pod phase（阶段）

Pod的`status`字段使`PodStatus`对象，具有`phase`字段。Pod的阶段是Pod在其生命周期中简单，高级摘要。阶段不是对容器或Pod状态的全面观察汇总，也不是一个综合状态机。

Pod的`phase`值的数量和含义受到严密保护。除了这里记录的内容外，没有任何关于具体给定相位值的Pod的假设。

| Value              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `Pending`          | Pod已被Kubernetes系统接收，但稍微创建一个或多个容器镜像。这包括计划之前的时间以及通过网络下载镜像所花费的时间. |
| `Running`          | Pod已绑定到节点，并且易创建所有容器。至少有一个容器仍在运行，或者正在启动或重新启动。 |
| `Succeeded`        | Pod中的所有容器都已经成功终止。并且不会重新启动。            |
| `Failed`           | Pod中的所有容器都已经终止，并且至少有一个容器已经终止失败。也就是说，容器要么非零状态退出。要么被系统终止. |
| `Unknown`          | 由于某种原因，无法获得Pod的状态，这通常是由于Pod的主机通信时出错。 |
| `Completed`        | Pod已经运行完成，因为没有什么可以让他们继续运行，例如，完成工作。 |
| `CrashLoopBackOff` | 这意味着Pod中的一个容器已意外退出，并且即使在容器后由于重启策略也可能具有非零错误代码。 |



### Pod 状态

Pod有一个PodStatus，它有一个PodConditions数组。PodCondition数组的每个元素都有六个可能的字段：

-  `lastProbeTime` 字段提供了上次探测Pod条件的时间戳。
-  `lastTransitionTime` 字段提供Pod最后一个状态转换到另一个状态的时间戳。
-  `message` 字段使人类可读的消息，只是有关状态的详细信息。
-  `reason`字段是状态最后一次转换的原因。
-  `status` 字段是一个字符串，可能的值是 `True`, `False`, 和`Unknown`.
-  `type` 字段是一个字符串，是一个包含以下可能值的字符串。
  - `PodScheduled`: Pod已被安排到一个节点；
  - `Ready`: Pod能够提供请求，应该添加到所有匹配的`service`的负载均衡池中；
  - `Initialized`: 所有`init`容器都成功启动。
  - `Unschedulable`: 调度程序现在无法调度Pod，例如缺乏资源或其他限制。
  - `ContainersReady`: Pod中的所有容器都准备好了。



### 容器探针

探针是由容器上的`kubelet`定期执行的诊断。为了执行诊断，`kubelet`调用容器实现的Handler。有三种类型的处理程序：

- `ExecAction`：在Container内执行指定命令。如果命令以状态码0退出，则认为诊断成功。
- `TCPSocketAction`：对指定端口上的容器的IP地址执行检查。如果端口打开，则诊断被认为是成功的。
- `HTTPGetAction`：对指定端口和路径上的容器的IP地址执行HTTP Get请求。如果相应的状态码大于或等于200且小于400，则认为诊断成功。

每个探针都有三个结果之一：

- **Success**：通过了诊断。
- **Failure**：未通过诊断。
- **Unknown**：诊断失败，因此不采取任何措施。

在运行容器时，`kubelet`可以选择性地执行和响应两种探测器：

- **livenessProbe**：指示容器是否正在运行。如果活动探测失败，则`kubelet`会杀死容器，并且容器将受其重启策略的约束。如果容器未提供活动探测，则默认状态为`Success`。
- **readinessProbe**：指示容器是否已准备好为请求提供服务，如果准备就绪探测失败，则端点控制器会从与Pod匹配的所有服务中的端点中删除Pod的IP地址。初始延迟之前的默认准备状态是`Failure`，如果容器未提供就绪状态探测，则默认状态为`Success`。

#### 什么时候应该使用活动或准备探针?

如果容器中进程在遇到问题或变得不健康时能够自行崩溃。则不一定需要活动探针。`kubelet`将提供Pod的重启策略自动执行正确的操作。

如果您希望在探测失败时杀死并重新启动Container，则指定活动探测，并指定`restartPolicy`Always或OnFailure。 

如果您只想在探测成功时开始向Pod发送流量，请指定准备探测。在这种情况下，准备情况探测可能与活动探测相同，但规范中存在准备探测意味着Pod将在不接收任何流量的情况下启动，并且仅在探测开始成功后才开始接收流量。如果Container需要在启动期间处理大型数据，配置文件或迁移，请指定就绪性探针。 

如果您希望Container能够自行维护，您可以指定一个就绪探针，用于检查特定于就绪状态的端点，该端点与活动探针不同。 

请注意，如果您只想在删除Pod时排除请求，则不一定需要准备探测; 在删除时，无论准备情况探测是否存在，Pod都会自动将其置于未准备状态。Pod在等待Pod中的容器停止时仍处于未准备状态。 



### 容器状态

一旦Pod被调度程序分配给节点，`kubelet`就开始使用容器运行时创建容器。容器由三种可能的状态：等待，运行和终止。要检查容器的状态，可以使用`kubectl describe pod [POD_NAME]`。显示Pod中每个容器的状态。

- `Waiting`：容器的默认状态。如果容器未处于`Running`或`Terminated`状态，则它处于`Waiting`状态。处于`Waiting`的容器仍然运行其所需的操作，如拉取镜像，应用`Secrets`等。随着此状态，将显示有关状态的消息和原因（`Reason`）以提供更多信息。

  ```yaml
  ...
    State:          Waiting
     Reason:       ErrImagePull
    ...
  ```

  

- `Running`：指出容器正在执行而没有问题。一旦容器进入`Running`状态，就会执行`postStart`钩子（如果有）。 

  ```yaml
     ...
        State:          Running
         Started:      Wed, 30 Jan 2019 16:46:38 +0530
     ...
  ```

- `Terminated`：表示容器已经完成执行并已停止运行。容器在成功完成执行或由于某种原因失败时进入此容器。无论如何，会显示原因和退出代码，以及容器的开始和结束时间。在容器进入`Terminated`之前，执行`preStop`钩子（如果有）。

  ```yaml
     ...
        State:          Terminated
          Reason:       Completed
          Exit Code:    0
          Started:      Wed, 30 Jan 2019 11:45:26 +0530
          Finished:     Wed, 30 Jan 2019 11:45:26 +0530
      ...
  ```



### Pod readiness gate

为了通过注入额外的反馈或信号来增加Pod准备的可扩展性`PodStatus`，Kubernetes 1.11引入了一个名为Pod ready ++的功能。您可以使用新的字段`ReadinessGate`中`PodSpec`指定波德准备进行评估附加条件。如果Kubernetes在`status.conditions`Pod 的字段中找不到这样的条件，则条件的状态默认为“ `False`”。以下是一个例子：

```yaml
Kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready  # this is a builtin PodCondition
      status: "True"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"   # an extra PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```

新Pod条件必须符合Kubernetes标签密钥格式。由于该`kubectl patch`命令仍然不支持修补对象状态，因此必须`PATCH`使用其中一个KubeClient库通过操作注入新的Pod条件。

随着新Pod条件的引入，**只有** 当以下两个语句都成立时，**才会**评估Pod是否就绪：

- Pod中的所有容器都已准备就绪。
- 指定的所有条件`ReadinessGates`均为“ `True`”。

为了便于对Pod准备评估进行此更改，`ContainersReady`引入了一个新的Pod条件 来捕获旧的Pod `Ready`条件。

### 重启策略

Pod Spec有一个`restartPolicy`字段，可能包含`Always`，`OnFailure`和`Never`。默认值为`Always`。`restartPolicy`适用于Pod中的所有容器。`restartPolicy`仅指同一节点上的`kubelet`重新启动容器。由`kubelet`重新启动的一退出容器将以指数回退延迟重新启动，上限五分钟，并在成功执行十分钟后重置。一旦绑定到节点，Pod永远不会弹回到其他节点。



### Pod 寿命

一般来说，Pod不会消失，直到有人删除它们。这可能是人或控制器。此规则的唯一例外是，具有成功或失败超过一段时间的阶段的Pod将过期并自动删除。

有三种类型的控制器可供选择：

- 使用Job for Pod预期终止，例如，批量计算。Job仅适用于`restartPolicy`为`OnFailure`或`Never`的Pod。
- 对不希望终止的Pod（例如web服务器）使用`ReplicationController`，`ReplicaSet`或`Deployment`。`ReplicationController`仅适用于`restartPolicy`为`Always`的Pod。
- 使用需要为每台计算机运行一个Pod的`DeamonSet`，因为他们提供特定于计算机的系统服务。

三种类型的控制器都包含`PodTemplate`。建议创建适当的控制器并让他们创建Pod，而不是直接创建Pod。这是因为单独的Pod不能容忍机器故障，但控制器可以。

如果节点死亡或与集群指定其他部分断开连接，Kubernetes会应用策略将死亡的节点上的所有Pod的`phase`设置为`Failed`。、

### 例子

#### 高级活动探测示例

活动探测由kubelet执行，因此所有请求都在kubelet网络名称空间中进行。

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - args:
    - /server
    image: k8s.gcr.io/liveness
    livenessProbe:
      httpGet:
        # when "host" is not defined, "PodIP" will be used
        # host: my-host
        # when "scheme" is not defined, "HTTP" scheme will be used. Only "HTTP" and "HTTPS" are allowed
        # scheme: HTTPS
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      timeoutSeconds: 1
    name: liveness
```

#### 示例状态

- Pod正在运行并有一个Container。容器退出成功。
  - 记录完成事件。
  - 如果`restartPolicy`是：
    - Always：重启容器; Pod `phase`保持运行状态。
    - OnFailure：Pod `phase`成功。
    - Never：Pod `phase`成功。
- Pod正在运行并有一个Container。容器退出失败。
  - 记录失败事件。
  - 如果`restartPolicy`是：
    - Always：重启容器; Pod `phase`保持运行状态。
    - OnFailure：重启容器; Pod `phase`保持运行状态。
    - Never：Pod `phase`变为失败。
- Pod正在运行并有两个容器。容器1出现故障。
  - 记录失败事件。
  - 如果`restartPolicy`是：
    - Always：重启容器; Pod `phase`保持运行状态。
    - OnFailure：重启容器; Pod `phase`保持运行状态。
    - Never：不要重启容器; Pod `phase`保持运行状态。
  - 如果Container 1未运行，并且Container 2退出：
    - 记录失败事件。
    - 如果`restartPolicy`是：
      - Always：重启容器; Pod `phase`保持运行状态。
      - OnFailure：重启容器; Pod `phase`保持运行状态。
      - Never：Pod `phase`变得失败。
- Pod正在运行并有一个Container。容器耗尽内存。
  - 容器终止失败。
  - 记录OOM事件。
  - 如果`restartPolicy`是：
    - Always：重启容器; Pod `phase`保持运行状态。
    - OnFailure：重启容器; Pod `phase`保持运行状态。
    - Never：记录失败事件; Pod `phase`变得失败。
- Pod正在运行，磁盘已经死亡。
  - 杀死所有容器。
  - 记录适当的事件。
  - Pod `phase`变得失败。
  - 如果在控制器下运行，Pod将在其他位置重新创建。
- Pod正在运行，其节点已分段。
  - 节点控制器等待超时。
  - 节点控制器将Pod设置`phase`为Failed。
  - 如果在控制器下运行，Pod将在其他位置重新创建。

## Init容器



### 认识 Init 容器

Pod可以有多个容器在其中运行应用程序，但它也可以有一个或多个Init容器，这些容器在启动应用程序之前运行。

Init容器与常规容器完全相同，除了：

- 它们总是运行到完成。
- 每个必须在下一个启动之前成功完成。

如果Pod的Init容器失败，Kubernetes重复重启Pod，直到Init容器成功。但是，如果Pod的`restartPolicy`为Never，则不会重新启动。

要将容器指定为Init容器，将PodSpec上的`initContainers`字段添加为应用容器数组旁边的Conrainer类型的JSON数组。init容器的状态在`.status.initContainerStatuses`，字段中作为容器状态的数组返回。



### Init 容器可以用于什么

由于Init容器具有来自应用容器的单独镜像，因此它们对于启动相关代码具有一些优势：

- 出于安全原因，它们可以包含并运行不希望包含在应用容器镜像中的实用程序。
- 它们可以包含应用程序镜像中不存在的用于设置程序或自定义代码。例如，在安装过程中无需使用`sed`,`awk`,`python`或`dig`等工具制作来自其他镜像的镜像。
- 应用程序镜像构建器和部署可以独立工作，而无需共同构建单个应用程序镜像。
- 它们使用Linux namespaces，以便它们从应用程序容器中获得不同的文件系统视图。因此，它们可以访问应用容器无法访问的`Secrets`。
- 它们在任何应用程序容器启动之前运行完成，而应用程序容器并行运行，因此Init容器提供了一种简单的方式来阻止或延迟应用容器启动，直到满足一些前置条件为止。



### 详细的行为

在Pod启动期间，初始化网络和卷后，Init容器将按照顺序启动。每一个容器必须在下一个容器启动之前成功退出。如果容器由于运行时无法启动或因故障退出，则根据Pod的`restartPolicy`重试。但是，如果Pod`restartPolicy`设置为Always，则Init容器将使用`RestartPolicy`OnFailure。

在所有Init容器都成功之前，Pod无法就绪。Init容器上的端口不在服务下聚合。正在初始化的Pod处于`Pending`状态，但应该具有`Initializing`设置为true的条件。



如果从新启动Pod，则必须再次执行所有Init容器。

Init容器规范的更改仅限于容器镜像字段。更改Init容器字段相当于重启Pod。

由于Init容器可以重新启动，重试或重新执行，因此Init容器代码应该是幂等的。特别是，写入`EmptyDirs`上的文件的代码应该准备好输出文件已经存在的可能性。

Init容器具有应用容器的所有字段。但是，Kubernetes禁止使用`readinessProbe`，因为Init容器无法定义与完成不同的准备情况。这在验证期间强制执行。

在Pod上使用`activeDeadlineSeconds`在容器上使用`libenessProbe`以防止Init容器永远失败。

Pod中每个应用程序和Init容器的名称必须是唯一的；任何与另一个名称共享的容器都会引发验证错误。

#### 资源

鉴于Init容器的排序和执行，适用以下资源适用规则：

- 在所有Init 容器上定义的任何特定资源请求或限制的最有效的Init请求/限制。
- Pod对资源的有限请求/限制是更高的：
  - 所有应用容器资源请求/限制的总和
  - Init有效的资源请求/限制
- 调度是基于有效的请求/限制完成的，这意味着Init容器可以保留在Pod生命周期内使用的初始化资源
- Pod的有效Qos是Init容器和app容器的Qos。

根据有效的Pod请求和限制应用配额和限制。

Pod级别cgroup基于有效的Pod请求和限制，与调度程序相同。

#### Pod重启原因

由于以下原因，Pod可以重新启动，导致重启执行Init容器：

- 用户更新PodSpec，导致Init容器镜像发生更改。应用容器镜像更改仅重启应用程序容器
- Pod基础架构容器重新启动。这种情况并不常见，必须由对节点有root访问权限的人员完成。
- Pod中的所有容器会终止，而`restartPolicy`设置为Always，强制重新启动，并且Init容器完成记录由于垃圾回收而丢失。

## Pod Preset

### 了解Pod Preset

`Pod Preset`是一种API资源，用于在创建时将其他运行时需求注入Pod。可以使用`labbel selector`指定应用给定`Pod Preset`的Pod。

使用`Pod Preset`允许模板作者不必显式提供每个Pod的所有信息。这样使用特定服务的Pod模板的作者不需要互道有关服务的所有详细信息。

### 如何工作

Kubernetes提供了一个准入控制器（`PodPresets`），启用后，将PodPresets应用于传入的Pod创建请求。发生Pod创建请求时，系统会执行以下操作：

1. 检索可供使用的所有`PodPresets`
2. 检查任何`PodPreset`的标签选择器是否与正在创建的Pod上的标签匹配
3. 尝试将`PodPreset`定义的各种资源合并到正在创建的Pod中
4. 出错时，抛出一个记录Pod上合并错误的事件，并创建Pod而不从`PodPreset`注入任何资源。
5. 注释生成的修改狗的Pod规范，以指示它已被`PodPreset`修改。注释的格式：`podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"`

每个Pod可以匹配零个或多个PodPresets；并且每个`PodPresets`可以应用于零个或多个Pod。当`PodPreset`应用与一个或多个Pod时，Kubernetes会修改PodSpec。对于`Env`，`EnvFrom`和`VolumeMounts`的更改，Kubernetes修改Pod中的所有容器的容器规范；对于`Volume`的更改，Kubernetes修改Pod Spec。

## Disruptions(中断)

### 自愿和非自愿终端

在有人（一个人或一个控制器）摧毁他们，或者存在不可避免的硬件或系统软件错误之前，Pod不会消失。

我们将这些不可避免的案例成为对应用程序的非自愿中断。例如：

- 节点和物理机的硬件故障
- 集群管理员错误地删除了VM（实例）
- 云提供商或虚拟机管理程序故障使虚拟机消失
- Kernel panic（内核恐慌）
- 由于集群网络隔离，节点从集群中消失
- 由于节点资源不足而导致Pod被驱逐

除资源不足外，大多数用户都应熟悉所有这些条件；我们将其他案例称为自愿中断。其中包括应用程序所有者启动的操作和集群管理员启动的操作：

- 删除管理Pod的`deployment`或其他控制器
- 更新`deployment`的Pod模板导致的重启
- 直接删除一个Pod，例如意外

集群管理员操作包括：

- 排除节点进行修复或升级
- 从集群中删除节点以缩小集群
- 从节点中删除Pod以允许其他内容合适该节点

这些操作可以由集群管理员直接执行，也可以由集群管理员或集群主机提供商的自行运行。

### 处理中断

以下是一些缓解非自愿中断的方法:

- 确保Pod请求所需的资源
- 如果需要更高的可用性，请复制应用程序
- 为了运行复制的应用程序时后的更高的可用性，可以跨机架或区域分布应用程序

资源终端的频率各自不同。在基本的Kubernetes集群上，根本没有自愿中断。但是，集群管理员或托管服务提供商可能会运行一些导致资源终端的其他服务。