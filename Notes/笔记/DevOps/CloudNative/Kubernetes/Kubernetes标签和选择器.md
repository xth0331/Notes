# Kubernetes标签和选择器

Labels（标签）是附加到对象的键值对，例如Pod。标签旨在用于指定UI用户有意义且相关的对象的标识属性，但不直接暗示核心系统的语义。标签可用于组织和选择对象的子集。标签可以在创建是附加给对象，随后可以随时添加和修改。每个对象都可以定义一组键值对标签。每个Key对于给定对象必须是唯一的。

```json
"metadata": {
    "labels": {
        "key1": "value1",
        "key2": "value2"
	}
}
```

标签允许高级的查询和监控，非常适合在UI和CLI中使用。应使用注释记录非识别信息。

## 动机

标签使用户能够以松耦合的方式将他们自己的组织结构映射到系统对象，而不需客户端存储这些映射。

服务部署和批处理流水线通常是多维实体，管理通常需要跨领域的操作，这打破了严格的层次化表示的封装，特别是由基础设施而不是用户确定的层次结构。

示例标签：

- `"release": "stable"`,`"release": "canary"`
- `"environment": "dev"`, `"environment": "qa"`, `"environment": "production"`
-  `"tier": "frontend"`, `"tier": "backend"`,`"tier": "cache"`
- `"partition": "customerA"`, `"partition": "customerB"`
- `"track": "daily"`, `"track": "weekly"`

可以自由地指定自己的约定。记住，标签Key对于给定对象必须是唯一的。



## 标签选择器

与名称和UID不同，标签不提供唯一性。通常，我们希望许多对象携带相同的标签。

通过*标签选择器*，客户端/用户可以识别一组对象。标签选择器是Kubernetes中的核心分组原语。

目前，API支持两种类型的选择：*基于相等，*和*基于集的*。标签选择器可以由逗号分隔的多个*要求*组成。在多个要求的情况下，必须满足所有要求，因此逗号分隔符充当逻辑*AND*（`&&`）运算符。

空或非指定选择器的语义取决于上下文，使用选择器的API类型应记录它们的有效性和含义。

> **注意：**对于某些API类型（例如ReplicaSet），两个实例的标签选择器不得在命名空间内重叠，或者控制器可以将其视为冲突的指令，并且无法确定应存在多少副本。

### *基于相等的*要求

*基于相等*或*不相等的*要求允许按标签键和值进行过滤。匹配对象必须满足所有指定的标签约束，尽管它们也可能有其他标签。三种运营商都承认`=`，`==`，`!=`。前两个代表*相等*（简单地说是同义词），而后者代表*不相等*。例如：

```
environment = production
tier != frontend
```

前者选择密钥等于`environment`和值等于的所有资源`production`。后者选择密钥等于`tier`和值不同的`frontend`所有资源，以及没有带`tier`密钥标签的所有资源。可以过滤使用逗号运算符`production`排除的资源`frontend`：`environment=production,tier!=frontend`

基于相等的标签要求的一种使用场景是Pods指定节点选择标准。例如，下面的示例Pod选择标签为“ `accelerator=nvidia-tesla-p100`”的节点。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

### *基于集合的*要求

*基于集合的*标签要求允许根据一组值过滤密钥。三种运营商的支持：`in`，`notin`和`exists`（仅密钥标识符）。例如：

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

第一个示例选择Key等于`environment`和值等于`production`或的所有资源`qa`。第二个示例选择Key等于`tier`和除了`frontend`和`backend`之外的值的所有资源，以及没有带`tier`密钥标签的所有资源。第三个例子选择所有资源，包括带Key`partition`的标签; 没有检查值。第四个示例选择没有带Key`partition`的标签的所有资源; 没有检查值。类似地，逗号分隔符充当*AND*运算符。

*基于集合的*需求可以与*基于相等的*需求相结合。例如：`partition in (customerA, customerB),environment!=qa`。

## API

### LIST和WATCH过滤

LIST和WATCH操作可以指定标签选择器来过滤使用查询参数返回的对象集。这两个要求都是允许的（在此处显示为出现在URL查询字符串中）：

- *基于平等的*要求：`?labelSelector=environment%3Dproduction,tier%3Dfrontend`
- *基于集合的*要求：`?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

两种标签选择器样式都可用于通过REST客户端列出或查看资源。例如，靶向`apiserver`与`kubectl`和使用*基于平等-*一个可写：

```shell
kubectl get pods -l environment=production,tier=frontend
```

或使用*基于集合的*要求：

```shell
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

如前所述*，基于集合的*要求更具表现力。例如，他们可以在值上实现*OR*运算符：

```shell
kubectl get pods -l 'environment in (production, qa)'
```

或限制负匹配通过*存在*操作者：

```shell
kubectl get pods -l 'environment,environment notin (frontend)'
```

### 在API对象中设置引用

某些Kubernetes对象（例如`services`和`replicationcontrollers`）也使用标签选择器来指定其他资源集，例如pod。

#### Service和ReplicationController

`service`使用标签选择器定义目标的一组pod 。类似地，`replicationcontroller`应该管理的pod的数量也用标签选择器定义。

两个对象的标签选择器在使用映射定义`json`或`yaml`文件中定义，并且仅支持*基于相等的*需求选择器：

```json
"selector": {
    "component" : "redis",
}
```

要么

```yaml
selector:
    component: redis
```

这个选择器（分别以`json`或`yaml`格式）相当于`component=redis`或`component in (redis)`。

#### 支持基于集合的需求的资源

较新的资源，如[`Job`](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/)，[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)，[`Replica Set`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)，和[`Daemon Set`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)，支持*基于集合的*要求也是如此。

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

`matchLabels`是`{key,value}`的。一个单一的`{key,value}`在`matchLabels`相当于`matchExpressions`的元素，其`key`字段是“键”，则`operator`是“以”和`values`阵列仅包含“值”。`matchExpressions`是一个pod选择器要求列表。有效的运算符包括In，NotIn，Exists和DoesNotExist。在In和NotIn的情况下，设置的值必须是非空的。所有的要求，从两者`matchLabels`和`matchExpressions`AND一起 - 他们必须满足，以匹配。

#### 选择节点集

用于选择标签的一个用例是约束pod可以调度的节点集。

