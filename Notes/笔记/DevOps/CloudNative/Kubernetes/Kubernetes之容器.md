# Kubernetes之容器

## 镜像

可以创建Docker镜像并将其推送到`registry`，然后在Kubernetes`pod`中引用它。

容器的`image`属性支持与docker命令相同的语法，包括私有`registry`和`tag`。



### 更新镜像

默认的拉取策略是`IfNotPresent`，这会导致`kubelet`跳过拉取镜像（如果镜像已存在）。如果想总是拉取，可以执行以下操作之一：

- 将容器的`imagePullPolicy`设置为`Always`。
- 省略`imagePullPolicy`并使用`:latest`作为图像的标记（tag）。
- 省略`imagePullPolicy`和要使用的镜像标记。
- 启用`AlwaysPullimages`准入控制器。

> 注意，应该避免使用`:latest`tag。



## 容器环境变量

### 容器环境

Kubernetes Container环境为容器提供了几个重要资源：

- 文件系统，是image和一个或多个卷的组合。
- 有关Container本身的信息。
- 有关集群中其他对象的信息。

#### 容器信息

Container的主机名是运行Container的Pod的名称。它可以通过`hostname`命令或`libc`中的`gethostname`函数调用获得。

Pod名称和Namespace可通过`download API`作为环境变量使用。

Pod定义中的用户定义环境变量也可用于容器，Docker镜像中静态指定的任何环境变量也是如此。

#### 集群信息

创建容器是运行的所有服务的列表可作为环境变量用于该容器，这些环境变量与`docker line`的语法相匹配。

对于名为`foo`的`service`，映射到名为`bar`的容器，将定义以下变量：

```bash
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port eht service is running on>
```

`service`具有专用IP地址，如果启用了DNS差价，则可以通过DNS使用容器。

## Runtime Class

`RuntimeClass`是一个`alpha`特征，用于选择用于运行`pod`容器的容器运行时配置。

### 建立

作为早期的`alpha`功能，必须采取一些额外的设置步骤才能使用`RuntimeClass`。

1. 启用`RuntimeClass`功能
2. 安装`RuntimeClass` CRD
3. 在节点上配置CRI实现
4. 创建相应的`RuntimeClass`资源



### 用法

为集群配置`RuntimeClass`后，使用他们非常简单，在Pod规范中定义`runtimeClassName`例如：



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```

这将使`kubectl`使用指定的`RuntimeClass`来运行此pod。如果指定的`RuntimeClass`不存在，或者CRI无法运行相应的处理程序，则pod将进入`Failed`阶段。查找错误消息的相应事件。



如果未指定`runtimeClassName`则使用默认的RuntimeHandler，这相当于禁用`RuntimeClass`功能。



##  容器生命周期钩子

类似许多具有生命周期钩子的编程语言框架，例如`Angular`Kubernetes为容器提供了生命周期钩子。钩子使容器能能够了解其管理周期中的时间，并在执行相应的生命周期钩子时运行在处理程序中实现的代码。

### Container hooks

这里有两个暴露给容器的钩子：

- `PostStart`

  在创建容器后立即执行此钩子。但是，无法保证钩子将在容器`ENTRYPOINT`之前执行。没有参数传递给处理程序。

- `PreStop`

  由于API请求或管理时间（例如活动探测失败，抢占，资源征用等），在容器终止前立即调用此钩子。如果容器已处于已终止或完成状态，则对`perStop`钩子的调动将失败。它是阻塞的，意味着他是同步的，所以它必须在删除容器的调用之前完成。没有参数传递给处理程序。

#### 钩子处理程序实现

> 容器可以通过实现和注册该钩子的处理程序来访问钩子。可以为容器实现两种类型的钩子处理程序。

- Exec： 执行特定命令，比如，`per-stop.sh`，在容器内的`cgroups`和`namespaces`。命令消耗的资源将根据容器计算。
- HTTP：对容器上的特定`endpoint`执行HTTP请求。



#### 钩子处理程序执行

调用容器生命周期管理钩子时，Kubernetes管理系统会在为钩子注册的容器执行处理程序。

钩子处理程序调用在包含容器的Pod的上下文是同步的。这意味着对于`PostStart`钩子，容器`entrypoint`和钩子异步激活。但是，如果钩子运行或挂起太长时间，则容器无法达到运行状态。

`PerSrop`钩子的行为类似，如果钩子在执行期间挂起，则Pod阶段将保持`Terminating`转台，并在终止的`FinallyGracePriodSecounds`结束后被终止。如果`PostStart`或`PreStop`钩子失败，则会终止容器。

用户赢使其钩子处理程序尽可能轻量级。但是，有些情况，长时间运行的命令是有意义的，例如在停止容器之前保存状态。

#### 钩子交付保证

钩子交付至少需要一次，这意味着对于任何给定的事件，可以多次调用钩子，例如`PostStart`或`PreStop`。由钩子实现来正确处理这个问题。

通常，只进行单次交付。例如，如果HTTP钩子接收器关闭且无法获取流量，则不会尝试重新发送。

然而，在一些情况下，可能会发生双重递送。例如，如果一个`kubelet`在发送一个钩子的过程中从新启动，那么在`kubelet`重新启动后可能会重新发送一次。

#### 调试钩子处理程序

在`Pod`实践中不公开钩子处理程序的日志。如果处理程序由于某种原因，它会广播一个时间。对于`PostStart`，这是`FiledPostStartHook`事件，对于`PreStop`，这是`FiledPreStopHook`事件。可以通过`kubectl describe pod <POD_NAME>`。

