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

