# Kubernetes对象之namespace



Kubernetes支持由同一物理集群支持的多个虚拟集群。这些虚拟集群成为`namespace`，

## 何时使用多个namespace

namespace旨在用于多个用户分布在多个团队或项目中的环境中。对于具有几个到几十个用户的集群，根本不需要创建或考虑namespace，当需要它们提供的功能时，请开始使用命名空间。

namespace提供名称范围。资源名称在namespace中必须是唯一的，而不是跨namespace。

namespace是一种在多个用户之间划分集群资源的方法（通过资源配额）。

在未来的Kubernetes版本中，默认情况下，同`namespace`中的对象将具有相同的访问控制策略。

没有必要使用多个`namespace`来分隔略有不同的资源，例如同一软件的不同版本：使用标签来区分同一`namespace`中的资源。

##使用namespace

### 创建namespace

创建`mynamespace-yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```

然后运行

```bash
kubectl create -f mynamespace.yaml
```

### 查看namespace

你可以列出集群中的`namespace`:

```bash
kubectl get namespace
NAME               STATUS   AGE
default            Active   126d
kube-public        Active   126d
kube-system        Active   126d
```

- `default` 没有`namespace`的对象的默认`namespace`
- `kube-system` Kubernetes 系统对象的`namespace`
- `kube-pubilc` 自动创建的`namespace`，并且所有用户都可以读取。主要用于集群使用，防止某些资源在整个集群中可见且公开读取。`namespace`的公共方面只是一个约定，不是要求。

### 指定namespace

要临时设置请求的命名空间，请使用 `--namespace`

For example:

```shell
kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
kubectl --namespace=<insert-namespace-name-here> get pods
```

### 设置namspace首选项

可以在该上下文中为所有后续kubectl命令永久保存命名空间.

```shell
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
kubectl config view | grep namespace:
```



## namespace和DNS

创建`service`时，它会创建相应的DNS条目。条目是`<service-name>.<namespace-name>.svc.cluster.local`，这意味着如果容器只是使用`<service-name>`，它将解析为`namespace`本地的服务。这对于在多个名称空间（如开发，分段和生产）中使用相同的配置非常有用。如果要跨命名空间访问，则需要使用完全限定的域名（FQDN）。

## 并非所有对象都在namespace中

大多数Kubernetes资源（如Pods，Services，replication controllers）都在`namespace`中，但是`namespace`资源本身并不在`namespace`中。并且低级资源（例如，node，persistentVolumes）不在任何`namespace`中。

查看哪些Kubernetes资源是否在`namespace`中。

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

