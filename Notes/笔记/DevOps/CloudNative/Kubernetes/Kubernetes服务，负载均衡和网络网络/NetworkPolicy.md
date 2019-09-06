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

