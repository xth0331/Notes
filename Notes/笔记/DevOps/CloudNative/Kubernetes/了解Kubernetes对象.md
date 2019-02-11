# 了解Kubernetes对象

Kubernetes对象是Kubernetes系统中的持久实体。Kubernetes使用这些实体来表示集群的状态。具体来说，他们可以描述：

- 正在运行的容器化应用程序（以及在那些节点上）
- 这些应用程序可用的资源
- 有关这些应用程序行为方式的策略，例如重新启动策略，升级和容错。

Kubernetes对象是一个“record of intent(意图记录)”-你创建对象，Kubernetes系统将不断努力确保对象存在。通过创建一个对象，可以有效地告诉Kubernetes系统希望集群的工作负载看起来像什么；这是您的集群**期望状态**。

操作Kubernetes对象 —— 五落实创建、修改、或者删除 —— 需要使用Kubernetes API。比如，当使用`kubectl`命令行时，CLI会执行必要的Kubernetes API调用，也可以在程序中直接调用Kubernetes API。

## 对象规范和状态

每个 Kubernetes对象都包含两个嵌套对象字段，用于控制对象的配置：对象`spec`和对象`status`。必须提供的`spec`(规范)描述了对象所需的状态 - 希望对象具有的特征。状态描述对象的实际状态，由Kubernetes系统提供和更新。在任何给定时间，Kubernetes控制平面都会主动管理对象的实际状态，以匹配您所提供的所需状态。

例如，Kubernetes `Deployment`是一个对象，可以表示在您的集群上运行的应用程序。创建`Deployment`时，可以设置`Deployment`的`spec`（规范）以指定希望运行应用程序的三个副本。

Kubernetes系统读取`Deployment`的`spec`（规范）并启动所需应用程序的三个实例 - 更新状态以符合规范。如果这些实例中的任何一个失败（状态改变），Kubernetes系统通过进行更正来响应`spec`（规范）和状态之间的差异 - 在这种情况下，启动替换实例。

## 描述Kubernetes对象

在Kubernetes中创建对象时，必须提供描述其所需状态的对象规范，以及有关该对象的一些基本信息（例如名称）。当使用KubernetesAPI创建对象时（直接或通过kubectl），该API请求必须在请求正文中包含该信息作为JSON。通常，在`.yaml`文件中向`kubectl`系统信息。`kubectl`在发出API请求时将信息转换为JSON。

这是一个示例yaml文件，显示Kubernetes部署所需的字段和对象规范：

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      lavels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

使用类似上面的 `yaml`文件创建`Deployment`的一种方式是在`kubectl`命令行中使用`kubectl create `命令，将`.yaml`文件作为参数传递：

```bash
kubectl create -f xxx.yaml --record
```

## Required Fields(必填字段)

在要创建的Kubernetes对象的`.yaml`文件中，需要为以下字段设置值：

- **apiVersion** - 正在使用哪个版本的Kubernetes API 来创建此对象。
- **kind** - 想要创建什么对象。
- **metadata** - 有助于唯一标识对象的数据，包括`name`字符串，`UID`和可选`namespace`(命名空间)。



还需要提供对象`spec`字段。对象`spec`的精确格式对于每个Kubernetes对象都是不同的，并且包含特定于该对象的嵌套字段。[Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)可以找到使用Kubernetes创建的所有对象的规范格式。