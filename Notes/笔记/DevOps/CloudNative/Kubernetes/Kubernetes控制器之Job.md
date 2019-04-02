# Kubernetes控制器之Job

Job创建一个或多个Pod，并确保指定数量的Pod成功终止。当Pod成功完成时，Job将跟踪成功完成的情况。达到指定数量的成功完成时，任务（Job）完成。删除作业将清理它创建的Pod。

一个简单的例子是创建一个Job对象，以便可靠地运行一个Pod直至完成。如果第一个Pod失败或被删除（例如由于节点硬件故障或节点重新引导），Job对象将启动一个新的Pod。

还可以使用Job并行运行多个Pod。

## 运行Job示例

下面是一个示例Job配置。它将π计算到2000个位置并将其打印出来。完成大约需要十秒。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

查看信息：

```bash
kubectl describe jobs/pi    # 查看jobs
kubectl get JOB-POD-NAME    # 查看job生成的Pod信息
kubectl logs PI-POD-NAME    # 查看结果
```

## Job Spec

与所有其他Kubernetes配置一样。Job需要`apiVersion`,`kind`和`metadata`字段。

Job还需要`.spec`部分。

### Pod Template

`.spec.template`是`.spec`中唯一必须的字段。

除了Pod的必填字段外，Job中的Pod模板还必须指定适当的标签和适当的重启策略。

`RestartPolicy`只允许等于`Never`或`OnFailure`.

### Pod Selector

`.spec.selecror`字段是可选的。在几乎所有情况下，都不应该指定它。

### Parallel Jobs

有三种主要类型的任务合适作为Job运行。



1. 非并行：
   - 通常，只有一个Pod启动，除非Pod失败。
   - 一旦Pod成功终止，Job就完成了。
2. 具有固定完成计数的并行作业：
   - 为 `.spec.completions`指定一个非零的正值。
   - Job代表整体任务，当1到 `.spec.completions`范围内的每个值都有一个成功的Pod时，作业就完成了。
3. 具有工作队列的并行作业:
   - 不要指定`.spec.completions`, 默认为 `.spec.parallelism`.
   - Pod必须在它们之间或外部服务之间进行协调，已确定每个应该工作的内容。例如，Pod可能会从工作队列中获得最多N个项目的。
   - 每个Pod独立地能够确定是否完成了所有对等，从而完成了整个Job。
   - 当Job中断任何Pod成功终止时，不会创建新的Pod。
   - 一旦至少一个Pod成功终止并且所有Pod终止，则成功完成。
   - 一旦任何Pod退出成功，其他任何Pod都不应该为此任务做任何工作或编写任何输出。他们都应该在退出的过程中。

对于*非并行*作业，您可以保留两者`.spec.completions`并`.spec.parallelism`取消设置。当两者都未设置时，两者都默认为1。

对于*固定完成计数*作业，您应设置`.spec.completions`所需的完成次数。您可以设置`.spec.parallelism`或保持未设置状态，默认为1。

对于*工作队列* Job，您必须保留未`.spec.completions`设置，并设置`.spec.parallelism`为非负整数。



#### 控制并行性

请求的parallelism（`.spec.parallelism`）可以设置为任何非负值。如果未指定，则默认为1.如果将其指定为0，则作业将有效暂停，直至其增加。

由于各种原因，实际并行度（在任何时刻运行的pod的数量）可能多于或少于请求的并行度：

- 对于*固定完成计数*作业，并行运行的实际pod数不会超过剩余完成数。更高的值`.spec.parallelism`被有效忽略。
- 对于*工作队列*作业，任何Pod成功后都不会启动新的Pod - 但是，允许剩余的Pod完成。
- 如果控制器没有时间做出反应。
- 如果控制器由于任何原因（缺少`ResourceQuota`，缺少许可等）而无法创建Pod，那么可能会有比请求更少的pod。
- 由于同一作业中过多的先前pod故障，控制器可能会限制新的Pod创建。
- 如果正常关闭Pod，则需要一段时间才能停止。

## 处理Pod和容器故障

Pod中的容器可能由于多种原因而失败，例如因为其中的进程退出时具有非零退出代码，或者容器因超出内存限制而被杀死等等。如果发生这种情况`.spec.template.spec.restartPolicy = "OnFailure"`，那么， Pod停留在节点上，但重新运行容器。因此，您的程序需要在本地重新启动时处理该情况，否则请指定`.spec.template.spec.restartPolicy = "Never"`。

整个Pod也可能由于多种原因而失败，例如当pod从节点上被踢出（节点被升级，重新启动，删除等），或者如果Pod的容器失败而且 `.spec.template.spec.restartPolicy = "Never"`。当Pod失败时，作业控制器启动一个新的Pod。这意味着您的应用程序需要在新pod中重新启动时处理该案例。特别是，它需要处理由先前运行引起的临时文件，锁，不完整的输出等。

请注意，即使你指定`.spec.parallelism = 1`和`.spec.completions = 1`和`.spec.template.spec.restartPolicy = "Never"`，同样的程序有时会两次启动。

如果您指定`.spec.parallelism`并且`.spec.completions`两者都大于1，那么可能会同时运行多个pod。因此，您的pod还必须容忍并发性。

### Pod退避失败策略

在某些情况下，由于配置中的逻辑错误，您需要在重试一定次数后使作业失败。为此，请设置`.spec.backoffLimit`为在将作业视为失败之前指定重试次数。默认情况下，退避限制设置为6.作业控制器重新创建与作业关联的失败窗格，指数后退延迟（10秒，20秒，40秒......）上限为6分钟。如果在作业的下一次状态检查之前没有出现新的失败Pod，则重置退避计数。

## 工作终止和清理

作业完成后，不再创建Pod，但也不会删除Pod。保持它们可以让您仍然查看已完成的pod的日志，以检查错误，警告或其他诊断输出。作业对象在完成后也会保留，以便您可以查看其状态。在注意到其状态后，用户可以删除旧作业。使用`kubectl`（例如`kubectl delete jobs/pi`或`kubectl delete -f ./job.yaml`）删除作业。使用时删除作业时`kubectl`，也会删除它创建的所有窗格。

默认情况下，作业将不间断地运行，除非Pod失败，此时作业将遵循上述 `.spec.backoffLimit`描述。终止作业的另一种方法是设置活动截止日期。通过将`.spec.activeDeadlineSeconds`作业字段设置为秒数来完成此操作。

`activeDeadlineSeconds`适用于工作，期间不管如何创建许多豆荚。一旦作业到达`activeDeadlineSeconds`，所有的豆荚的终止和工作状态将成为`type: Failed`与`reason: DeadlineExceeded`。

请注意，作业`.spec.activeDeadlineSeconds`优先于作业`.spec.backoffLimit`。因此，重试一个或多个失败的Pod的作业一旦达到指定的时间限制`activeDeadlineSeconds`，即使`backoffLimit`尚未到达，也不会部署其他Pod 。

例：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```



## 自动清理完成的作业

系统中通常不再需要完成的作业。将它们保留在系统中会给API服务器带来压力。如果作业由更高级别的控制器（如CronJobs直接管理，则 CronJobs可以根据指定的基于容量的清理策略清除作业。

### 完成工作的TTL机制

另一种自动清理已完成作业（`Complete`或者`Failed`）的方法是使用TTL控制器提供的TTL机制 来完成资源，方法是指定`.spec.ttlSecondsAfterFinished`作业的字段。

当TTL控制器清理作业时，它将级联删除作业，即删除其依赖对象（如Pod）和作业。请注意，删除作业时，将终止其生命周期保证，例如终结器。

例如：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

作业`pi-with-ttl`将有资格`100` 在完成后几秒钟自动删除。

如果该字段设置为`0`，则作业将有资格在完成后立即自动删除。如果未设置该字段，则TTL控制器完成后将不会清除此作业。



## 工作模式

Job对象可用于支持Pod的可靠并行执行。Job对象不是为支持密切通信的并行进程而设计的，这在科学计算中很常见。它支持并行处理一组独立但相关的*工作项*。这些可能是要发送的电子邮件，要呈现的帧，要转码的文件，要扫描的NoSQL数据库中的键的范围等等。

在复杂系统中，可能存在多组不同的工作项。在这里，我们只考虑用户想要一起管理的一组工作项 - *批处理作业*。

并行计算有几种不同的模式，每种模式都有优点和缺点。权衡是：

- 每个工作项的一个Job对象，而不是所有工作项的单个Job对象。后者适用于大量工作项目。前者为用户和系统创建了一些管理大量Job对象的开销。
- 创建的pod数等于工作项数，而每个Pod可以处理多个工作项。前者通常需要对现有代码和容器进行较少的修改。后者对于大量工作项目更好，原因与上一个项目类似。
- 有几种方法使用工作队列。这需要运行队列服务，并修改现有程序或容器以使其使用工作队列。其他方法更容易适应现有的集装箱化应用。

这里总结了权衡，第2列到第4列对应于上述权衡。模式名称也是示例和更详细描述的链接。

| 模式                        | 单个作业对象 | Pod比工作少？ | 使用app未经修改？ | 在Kube 1.1中有效吗？ |
| :-------------------------- | :----------- | :------------ | :---------------- | :------------------- |
| 工作模板扩展                |              |               | ✓                 | ✓                    |
| 具有Pod Per Work Item的队列 | ✓            |               | 有时              | ✓                    |
| 具有可变Pod计数的队列       | ✓            | ✓             |                   | ✓                    |
| 具有静态工作分配的单个作业  | ✓            |               | ✓                 |                      |

指定完成时`.spec.completions`，作业控制器创建的每个Pod都具有相同的[`spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status)。这意味着任务的所有pod将具有相同的命令行和相同的图像，相同的卷和（几乎）相同的环境变量。这些模式是安排pod处理不同事物的不同方式。

下表列出了所需的设置`.spec.parallelism`，并`.spec.completions`为每个模式。这里`W`是工作项的数量。

| 模式                        | `.spec.completions` | `.spec.parallelism` |
| :-------------------------- | :------------------ | :------------------ |
| 工作模板扩展                | 1                   | 应该是1             |
| 具有Pod Per Work Item的队列 | w ^                 | 任何                |
| 具有可变Pod计数的队列       | 1                   | 任何                |
| 具有静态工作分配的单个作业  | w ^                 | 任何                |

## 高级用法

### 指定您自己的pod选择器

通常，在创建Job对象时，不指定`.spec.selector`。创建作业时，系统默认逻辑会添加此字段。它选择一个不会与任何其他作业重叠的选择器值。

但是，在某些情况下，您可能需要覆盖此自动设置的选择器。为此，您可以指定`.spec.selector`作业。

这样做时要非常小心。如果您指定的标签选择器对于该作业的窗格不是唯一的，并且与不相关的窗格匹配，则可以删除不相关作业的窗格，或者此作业可能将其他窗格计为完成它，或者一个或两个作业可能拒绝创建Pod或运行完成。如果选择了非唯一选择器，那么其他控制器（例如ReplicationController）及其Pod也可能以不可预测的方式运行。Kubernetes不会阻止你在指定时犯错误`.spec.selector`。

以下是您可能希望使用此功能的示例。

说Job `old`已经在运行了。您希望现有Pod继续运行，但您希望其创建的其余Pod使用不同的pod模板，并使Job具有新名称。您无法更新作业，因为这些字段不可更新。因此，您删除工作`old`，但*离开它的吊舱运行*，使用`kubectl delete jobs/old --cascade=false`。在删除它之前，请记下它使用的选择器：

```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

然后使用名称创建一个新作业，`new`并明确指定相同的选择器。由于现有的Pod具有标签`job-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`，因此它们也由Job控制`new`。

您需要`manualSelector: true`在新作业中指定，因为您没有使用系统通常自动为您生成的选择器。

```
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

新的工作本身将有一个不同的uid `a8f3d00d-c6d2-11e5-9f87-42010af00002`。设置 `manualSelector: true`告诉系统您知道自己在做什么并允许这种不匹配。

## 备择方案

### Pod

当Pod正在运行的节点重新启动或失败时，该pod将终止并且不会重新启动。但是，Job将创建新的Pod来替换已终止的Pod。因此，我们建议您使用Job而不是裸Pod，即使您的应用程序只需要一个Pod。

### replication-controller

作业是复制控制器的补充。复制控制器管理不期望终止的Pod（例如Web服务器），并且Job管理预期终止的Pod（例如批处理任务）。

正如所讨论的Pod生命周期，`Job`是*仅*适用于具有豆荚`RestartPolicy`等于`OnFailure`或`Never`。（注意：如果`RestartPolicy`未设置，则默认值为`Always`。）

### 单个作业启动Controller Pod

另一种模式是单个Job创建一个Pod，然后创建其他Pod，作为这些Pod的一种自定义控制器。这允许最大的灵活性，但开始时可能有些复杂，并且与Kubernetes的集成度较低。

这种模式的一个例子是一个Job，它启动一个Pod，它运行一个脚本，然后启动一个Spark主控制器（参见[spark示例](https://github.com/kubernetes/examples/tree/master/staging/spark/README.md)），运行一个Spark驱动器，然后清理。

这种方法的一个优点是整个过程可以获得Job对象的完成保证，但可以完全控制创建Pod以及如何为其分配工作。

## Cron Jobs

您可以使用 `CronJob`来创建将在指定时间/日期运行的作业，类似于Unix工具`cron`。