# 网络策略

网络策略是允许Pod组与彼此其他网络端口通信的规范。

`NetworkPolicy`资源使用标签来选择Pod和定义规则，这些规则指定允许向所选的Pod发送哪些流量。



## 先决条件

网络策略由网络插件实现，因此必须使用支持`NetworkPolicy`的网络解决方案 - 只需创建资源而无需控制器来实现，将不起作用。

## 隔离和非隔离的Pod



默认情况下，Pod是非隔离的；他们接受任何来源的流量。

通过选择Pod的网络策略，Pod就被隔离了。一旦`namespace`中有任何`NetworkPolicy`选择特定的Pod，该pod将拒绝任何`NetworkPolicy`不允许的任何连接。（`namespace`中未被任何`NetworkPolicy`选中的其他Pod将继续接受所有流量。）

## NetworkPolicy资源

`NetworkPolicy`示例如下所示：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

除非选择的网络解决方案支持网络策略，否则将此发布到API服务器将不起作用。

强制字段：　与所有其他Kubernetes配置一样，`NetworkPolicy`需要`apiVersion`,`kind`和`metadata`字段。

**spec**: `NetworkPolicy`规范具有在给定`namespace`中定义特定网络策略所需的所有信息。

**podSelector**：每个`NetworkPolicy`都包含一个`podSelector`，用于选择策略适用的Pod分组。示例则略选择标签为`role = db`的Pod。一个空`podSelector`选择`namespace`中的所有Pod。

**policyTypes**：每个`NetworkPolicy`都包含一个`policyTypes`列表，其中可能包含`Ingress`，`Egress`或两者。`PolicyTypes`字段指定给定策略是否适用于对选定Pod的入口流量、出口流量，或两者。如果在`NetworkPolicy`上未指定`policyTypes`，那么默认情况下将始终设置出口。

**ingress**：每个网络策略可能包括一个白名单的`ingress`规则列表。每个规则都允许同时匹配`from`和`ports`部分的流量。示例策略包含一个规则，该规则匹配单个端口上的流量，来自三个源中的一个`ipblock`，第一个通过`ipblock`指定，第二个通过`namespaceSelector`指定，第三个通过`podSelector`指定。

**egress**：每个`NetworkPolicy`可以包括白名单出口规则列表。 每个规则都允许同时匹配`to`和`ports`部分的流量。示例策略包含单个规则，该规则将单个端口上的流量与`10.0.0.0/24`中的任何目标进行匹配。

那么：示例`NetworkPolicy`:

1. 隔离**default**`namespace`中`role=db`的Pod的入口和出口流量（如果他们还没有被隔离）。
2. `入口规则`允许连接到**default**`namespace`中的所有`role=db`Pod的TCP`6379`端口，从：
   - **default**`namespace`中任何标签为`role=frontend`的Pod。
   - 标签为`project=myproject`的任何Pod。
   - IP地址`172.17.0.0/16`和`172.17.1.0/24`范围内。
3. `出口规则`允许`namespace`为**default**中的任何标签`role=db`的Pod连接到CIDR 10.0.0.0/24的TCP5978端口。



##  `to` 和`from` 行为的selector

在`ingress`、`from`或`egress`、`to`部分可以指定四种选择器:

**podSelector**: 这将选择与 `NetworkPolicy` 相同`namespace`中的特定Pod，而`NetworkPolicy`应该被允许作为`ingress`源或`egress`目的。

**namespaceSelector**：这将选择应允许特定`namespace`的所有Pod作为`ingress`源或`egress`目的

**namespaceSelector** *和* **podSelector**: 指定`namespaceSelector`和`podSelector`的单个`to/from`条目选择特定`namespace`中的特定Pod。注意使用正确的YAML语法；这一策略：

```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
  ...
```

包含一个`from`元素，允许来自`namespace`中标签`role=client`，`user=alice`。但是这个：

```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
  ...
```

在`from`数组中包含两个元素，允许来自本地`namespace`中标有`role=client` 的Pod的连接，或来自本地任何`namespace`中标有`user=alice`的任何Pod的连接。



**ipBolck**: 这将选择特定的IP CIDR范围以用作入口源或出口目的。这些应该是集群外部IP，因为Pod IP存在时间短暂且随机产生。

集群的入口和出口机制通常需要重写数据包的源IP或目标IP。在发生这种情况下，不确定在`NetworkPolicy`处理之前还是之后发生，并且对于网络插件，云提供商，`Service` 实现等不同组合，其行为可能会有所不同。

在进入的情况下，这意味着在某些情况下，可以根据实际的源IP过滤传入的数据包，而在其他情况下，`NetworkPolicy`所作用的源IP则可能是`LoadBalancer`或Pod的节点。

对于出口，这意味着从Pod到被重写为集群外部IP的`Service`IP的连接可能不会受到基于ipBlock的策略约束。

## 默认策略

默认情况下，如果`namespace`不存在任何策略，则所有进出`namespace`的Pod流量都被允许。以下示例使您可以更改该`namespace`中的默认策略。



### 默认拒绝所有入口流量

您可以通过创建选择所有容器但不允许任何进入这些容器的入口流量的`NetworkPolicy`来为`namespace`创建`default`隔离策略。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: defualt-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

这样可以确保即使容器没有任何其他任何`NetworkPolicy`，也仍然可以被隔离。此策略不会更改默认的出口隔离行为。

### 默认允许所有入口流量

如果要允许所有流量进入某个`namespace`中的所有Pod（即使添加了导致某些Pod被视为隔离的策略），则可以创建一个策略来明确允许该`namespace`中的所有流量。

```yaml
apiVersion: netowrking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```

### 默认拒绝所有出口流量

可以通过创建选择所有容器但不允许来自这些容器的任何出口流量的`NetworkPolicy`来为`namespace`创建defualt egress隔离策略。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec: 
  podSelector: {}
  policyTypes:
  - Egress
```

这样可以确保即使某人被其他任何`NetworkPolicy`选择的Pod也不会被允许流出流量。此策略不会更改默认的`Ingress`隔离行为。

### 默认允许所有出口流量

如果要允许来自`namespace`中所有Pod的所有流量，则可以创建一条策略，该策略明确允许该`namespace`中的所有出口流量。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  egress:
  - {}
  policyType:
  - Egress
```

### 默认拒绝所有入口和所有出口流量

可以为`namespace`创建default策略，以通过在该`namespace`中创建以下`NetworkPolicy`来阻止所有入站和出站流量。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

这样可以确保即使没有任何其他`NetworkPolicy`选择的Pod也不会被允许进入或流出流量。

## SCTP支持

Kubernetes支持SCTP作为`NetworkPolicy`定义中的协议值作为`alpha`功能提供。要启用此功能，集群管理员需要在`apiserver`上启用`SCTPSuppotr`功能，例如`--feature-gates=SCTPSupport=true`。启用后，用户可以将`NetworkPolicy`

的`protocol`字段设置为`SCTP`。Kubernetes相应地位SCTP关联设置网络，就像TCP连接一样。

CNI插件必须在`NetworkPolicy`中将SCTP作为`protocol`支持。