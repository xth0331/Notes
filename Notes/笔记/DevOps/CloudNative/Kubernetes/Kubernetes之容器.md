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

