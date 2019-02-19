# 使用kubectl管理Kubernetes对象

## Kubernetes对象管理

`kubectl `命令行支持多种不同的方法来创建和管理。

### 管理技巧

> 应仅使用一种技术管理Kubernetes对象。对同一对象的混合和匹配会导致未定义的行为。



| 管理技术       | 操作        | 推荐环境 | 学习曲线 |
| -------------- | ----------- | -------- | -------- |
| 命令行         | Live Object | 开发项目 | 最低     |
| 对象配置       | 单个文件    | 生产项目 | 中等     |
| 声明式对象配置 | 文件目录    | 生产项目 | 最高     |

### 命令行

使用命令行操作是，用户直接在集群中的活动对象上操作。用户将操作以参数的方式提供给`kubectl`。

这是在集群中启动或运行一次性任务最简单的方法。由于命令行操作直接在活动对象上运行，因此它不提供之前配置的历史。

#### 例如

通过创建`deployment`对象来运行nginx容器的实例：

```bash
kubectl run nginx --image nginx 
```

使用不同的语法执行相同的操作：

```bash
kubectl create deployment nginx --image nginx
```

#### 权衡

与对象配置相比的优点：

- 命令简单易学，易记。
- 命令只需一个步骤就可以对集群进行更改。

与对象配置相比的缺点：

- 命令不与审核过程集成。
- 命令不提供与更改关联的审计跟踪。
- 除了活动之外，命令行不提供记录源。
- 命令不提供用于创建新对象的模板。

### 对象配置

在命令行对象配置中，kubectl命令指定操作（创建，替换等）,可选标志和至少一个文件名。指定的文件必须包含`YAML`或`JSON`格式的对象的完整定义。

#### 例如

创建配置文件中定义的对象：

```bash
kubectl create -f nginx.yaml
```

删除两个配置文件中定义的对象:

```bash
kubectl delete -f nginx.yaml -f redis.yaml
```

通过覆盖实时配置来更新配置文件中定义的对象：

```bash
kubectl replace -f nginx.yaml
```

#### 权衡

与命令形式相比的优点：

- 对象配置可以存储在诸如`git`的源码版本控制系统中。
- 对象配置可以与进程集成，例如在推送和审计跟踪之前查看更改。
- 对象配置提供了用于创建新对象的模板。

与命令形式相比的缺点：

- 对象配置需要对对象模式有基本的了解

- 对象配置需要编写YAML文件。



与声明式对象配置相比的优点：

- 对象配置相对简单，更易于理解。
- 1.5版本开始，命令式对象配置更成熟。

与声明对象配置相比的缺点：

- 命对象配置最适合文件，而不是目录。
- 活动搞对象的更新必须反映在配置文件中，否则下次更换时会丢失。

### 声明式对象配置

使用声明式对象配置时，用户对本地存储的对象配置文件进行操作，但是用户不定义要对文件执行的操作。`kubectl `自动检测每个对象的创建，更新和删除操作，这使得能够处理目录，其中可能需要不同对象的不同操作。

> 声明式对象配置保留其他编写者所做的更改，即使更改未合并会对象配置文件也是如此，这可以通过使用`pathch`API操作来金写入观察到的差异，而不是使用`replica`API操作来替换整个对象配置。

#### 例如

处理`configs`目录中的所有对象配置文件,并创建或修补活动对象。可以先查看要进行的更改，然后应用：

```bash
kubectl diff -f configs/
kubectl apply -f configs/
```

递归处理目录：

```bash
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

#### 权衡

与命令式对象配置相比的优点：

- 即使他们未合并回配置文件，也会保留直接对对象所做的更改。
- 声明式对象配置更改地支持对目录进行操作并自动检测每个对象的操作类型。

与命令式对象配置相比的缺点：

- 声明式对象配置更难以调试，并在意外时理解结果。
- 使用`diff`的部分更新会创建复杂的合并和修补操作。



## 使用命令管理Kubernetes对象

使用`kubectl`命令行工具中内置的命令可以快速创建，更新和删除Kubernetes对象。

### 如何创建对象

`kubectl`工具支持动词驱动的命令，用户创建一些最常见的对象类型。

- `run`： 创建一个新的`deployment`对象以在一个或多个`pod`中运行容器。
- `expose` : 创建一个新的`service`对象，以便在`pod`之间对流量进行负载均衡。
- `autoscale`：创建新的`autoscaler`对象以自动水平拓展控制器，例如`deployment`。

`kubectl `工具还支持由对象类型驱动的命令。

`create <objecttype> [<subtype>] <instancename>`

某些对象类型具有子类型，可在`create`命令中指定。例如，`service`对象有几个子类型，包括`ClusterIP`,`LoadBalance`和`NodePort`。这是一个使用子类型`NodePort`创建`service`的示例：

```shell
kubectl create service nodeport <myservicename>
```

在前面的例子中，`create service nodeport`命令被称为`create service`的子命令。

 ### 如何更新对象

`kubectl `支持一些常见更新操作的动词驱动命令。

- `scale`：通过更新控制器的副本数，水平缩放控制器以添加或删除`pod`。
- `annotate`： 在对象中添加或删除注释。
- `label`：在对象中添加或删除标签。

`kubectl`还支持由对象驱动的更新命令。

- `set  <field>`: 设置对象。

`kubectl`还支持直接实时更新对象的方法。

- `edit`： 通过在编辑器中打开其配置，直接编辑活动对象的原始配置。
- `patch`： 使用`patch`直接修改活动对象的特定字段。

### 如何删除对象

您可以使用该`delete`命令从群集中删除对象：

- `delete <type>/<name>`

> **注意：**您可以使用`kubectl delete`命令式命令和命令式对象配置。不同之处在于传递给命令的参数。要`kubectl delete`用作命令性命令，请将要删除的对象作为参数传递。这是一个传递名为nginx的Deployment对象的示例：

```bash
kubectl delete deployment/nginx
```

### 如何查看对象

有几个命令用于打印有关对象的信息：

- `get`：打印有关匹配对象的基本信息。使用`get -h`查看选项列表。
- `describe`：打印有关匹配对象的聚合详细信息。
- `logs`：为在Pod中运行的容器打印stdout和stderr。

### 使用`set`命令在创建之前修改对象

有些对象字段没有可在`create`命令中使用的标志。在一些案件中，可以使用的组合 `set`并`create`指定对象创建前场的值。这是通过将`create`命令的输出传递给 `set`命令，然后返回到`create`命令来完成的。这是一个例子：

```bash
kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run | kubectl set selector --local -f - 'environment=qa' -o yaml | kubectl create -f -
```

1. `kubectl create service -o yaml --dry-run`命令为服务创建配置，但将其作为YAML打印到stdout，而不是将其发送到Kubernetes API服务器。
2. `kubectl set selector --local -f - -o yaml`命令从stdin读取配置，并将更新的配置作为YAML写入stdout。
3. `kubectl create -f -`命令使用stdin提供的配置创建对象。

### 使用`--edit`修改之前创建的对象

您可以`kubectl create --edit`在创建对象之前对其进行任意更改。这是一个例子：

```bash
kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run > /tmp/srv.yaml
kubectl create --edit -f /tmp/srv.yaml
```

1. `kubectl create service`命令为服务创建配置并将其保存到`/tmp/srv.yaml`。
2. `kubectl create --edit`命令在创建对象之前打开配置文件以进行编辑。





## 使用配置文件管理Kubernetes对象

可以使用`kubectl`命令行工具以及使用`yaml`或`json`编写的对象配置文件来创建，更新和删除Kubernetes对象。



### 如何创建对象

可以使用`kubectl create -f`从配置文件创建对象。

- `kubectl create -f <filename|url>`

### 如何更新对象

> **警告：**使用该`replace`命令更新对象会删除配置文件中未指定的规范的所有部分。这不应该与规范部分由集群管理的对象一起使用，例如类型服务`LoadBalancer`，其中`externalIPs`字段独立于配置文件进行管理。必须将独立管理的字段复制到配置文件中以防止`replace`丢弃它们。

您可以使用`kubectl replace -f`根据配置文件更新活动对象。

- `kubectl replace -f <filename|url>`

### 如何删除对象

您可以使用`kubectl delete -f`删除配置文件中描述的对象。

- `kubectl delete -f <filename|url>`

### 如何查看对象

您可以使用它`kubectl get -f`来查看有关配置文件中描述的对象的信息。

- `kubectl get -f <filename|url> -o yaml`

`-o yaml`标志指定打印完整对象配置。使用`kubectl get -h`查看选项列表。

### 限制

的`create`，`replace`和`delete`命令工作得很好，当每个对象的配置完全确定并记录在它的配置文件。但是，当更新活动对象并且更新未合并到其配置文件中时，更新将在下次`replace` 执行时丢失。如果控制器（例如HorizontalPodAutoscaler）直接对活动对象进行更新，则会发生这种情况。这是一个例子：

1. 您可以从配置文件创建对象。
2. 另一个源通过更改某个字段来更新对象。
3. 您从配置文件中替换该对象。步骤2中其他来源所做的更改将丢失。

如果需要支持同一对象的多个编写器，则可以使用它 `kubectl apply`来管理对象。

### 在不保存配置的情况下从URL创建和编辑对象

假设您具有对象配置文件的URL。您可以 `kubectl create --edit`在创建对象之前用于更改配置。这对于指向可由读者修改的配置文件的教程和任务特别有用。

```bash
kubectl create -f <url> --edit
```

### 从命令式命令迁移到命令式对象配置

从命令式命令迁移到命令式对象配置涉及几个手动步骤。

将活动对象导出到本地对象配置文件：

   ```bash
kubectl get <kind>/<name> -o yaml --export > <kind>_<name>.yaml
   ```

从对象配置文件中手动删除状态字段。

对于后续对象管理，请`replace`专门使用。

   ``` bash
kubectl replace -f <kind>_<name>.yaml
   ```

### 定义控制器选择器和PodTemplate标签

> **警告：**强烈建议不要更新控制器上的选择器。

推荐的方法是定义一个仅由控制器选择器使用的单个不可变PodTemplate标签，没有其他语义含义。

示例标签：

```yaml
selector:
  matchLabels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
```

