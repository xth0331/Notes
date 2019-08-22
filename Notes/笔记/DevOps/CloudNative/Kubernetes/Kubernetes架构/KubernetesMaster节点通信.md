# Kubernetes Master节点通信

## 概述

本文对Master节点（确切说是apiserver）和Kubernetes集群之间的通信路径进行了分类。目的是为了让用户能够自定义他们的安装，对网络配置进行加固，使得集群能够在不可信的的网络上（或者在一个云服务商完全公共的IP上）运行。



## Cluster -> Master

所有从集群到master的通信路径都终止于`apiserver`(其他master组件没有被设计为可暴露远程服务)。在一个典型的部署中，`apiserver`被配置为在一个安全的HTTPS端口（443）上监听远程连接并启用一种或多种形式的客户端身份认证机制。一种或多种客户端身份认证机制应该被启用，特别是在允许匿名请求或`service account tokens`的时候。

应该使用集群的公共根证书开通节点，如此他们就能够基于有效的客户端凭据安全连接`apiserver`。例如：在一个默认的GCE部署中，客户端凭据以客户端证书的形式提供给`kubelet`。

想要连接到`apiserver`的Pods可以使用一个`service account`安全的进行连接。这种情况下，当Pods被实例化时Kubernetes将自动的把公共根证书和一个有效的不记名令牌注入到pod里。kubernetes `service`（所有namespaces中）都配置了一个虚拟IP地址，用于转发（通过`kube-proxy`）请求到`apiserver`的HTTPS endpoint。

Master组件通过非安全（没有加密或认证）端口和集群的`apiserver`通信。这个端口通过只在master节点的localhost接口暴露，这样，所有在相同机器上运行的master组件就能和集群的`apiserver`通信。一段时间以后，master组件将变为使用带身份验证和权限验证的安全端口。

这样的结果使得从集群（在节点上运行的nodes和pods）到master的缺省连接操作模式默认被保护，能够在不可信或公网中运行。

## Master -> Cluster

从master（apiserver）到看到集群有两桶通信路径。第一种是从`apiserver`到集群中每个节点上运行的`kubelet`进程。第二种是从`apiserver`通过它的代理功能到任何node、pod或者service。



### apiserver -> kubelet

从`apiserver`到`kubelet`的连接用户获取pods日志、连接（通过kubectl）运行中的pods,以及使用`kubelet`的端口转发功能。这些连接终止于`kubelet`的HTTPS endpoint。

默认的，`apiserver`不会验证`kubelet`的服务证书，这会导致连接遭到中间人攻击，因而在不可信或公共网络上是不安全的。

为了对这个连接进行认证，请使用`--kubelet-certificate-anthority`标记给`apiserver`提供一个根证书捆绑，用户`kubelet`的服务证书。

如果这样不可能，又要求避免在不可信的或公共的网络上进行连接，请在`apiserver`和kubelet之间使用SSH隧道。

最后，应该启用Kubelet用户认证和/或权限认证来保护`kubelet API`。

### apiserver -> nodes,pods,和services

从`apiserver`到node、pod或者service的连接默认为纯HTTP当时，因此既没有认证，也没有加密。他们能够通过给API URL中的node、pod或service名称添加前缀`https:`来进行安全的HTTPS连接。但他们既不会认证HTTPS endpoint提供的的证书，也不会提供客户端证书。虽然这样连接是加密的，但它不会提供任何完整性保证。这些连接目前还不能安全的在不可信的或公共的网络上运行。

### SSH隧道

Google Kubernetes Engine使用SSH隧道保护Master -> Cluster通信路径。在这种配置下，`apiserver`发起一个到集群中每个节点的SSH隧道(连接到在22端口监听的ssh服务)并通过这个隧道传输所有到kubelet、node、pod或者service的流量。这个隧道保证流量不会在集群运行的私有GCE网络之外暴露。