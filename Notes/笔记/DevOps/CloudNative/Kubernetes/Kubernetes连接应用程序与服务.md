# Kubernetes连接应用程序与服务

## 用于连接容器的Kubernetes模型

现在已经有了一个储蓄运行分复制应用程序，可以在网络上公开它。在讨论Kubernetes的网络方法之前，有必要将其与Docker的正常网络工作方式进行比较。

默认情况下，Docker使用`host-private`(*主机专用*)网络，因此容器只能在一台机器上与其他容器通信。为了使Docker容器能够跨界点通信，必须在机器自己的IP地址上分配端口，然后将端口转发或代理到容器。这显然意味着容器必须协调它们非常小心的使用哪些端口，或者必须动态分配端口。

跨多个开发人员协调端口非常困难，并且会将用户暴露在它们无法控制的集群级别问题之外。Kubernetes假设Pod可以与其他Pod通信，不管它们落在哪个主机上。我们为每个Pod提供了自己的`Cluster-Private-IP`(*集群专用IP*)，因此无需再Pod之间显式创建连接或将容器端口映射到主机端口。这意味着一个Pod中的容器都可以到达本地主机上彼此的端口，并且集群中所有Pod都可以在没有NAT的情况下看到彼此。

使用一个简单的Nginx服务器来演示概念证明。

## 将Pod暴露给集群

创建一个Nginx Pod，并注意它有一个容器端口规范：

`run-my-nginx.yaml:`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec: 
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec: 
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

这使得它可以从集群中的任何节点访问。检查Pod正在运行的节点：

```bash
kubectl apply -f ./run-my-nginx.yaml
kubectl get pods -l run=my-nginx -o wide
```

```bash
NAME                        READY   STATUS    RESTARTS   AGE    IP             NODE                    NOMINATED NODE   READINESS GATES
my-nginx-86459cfc9f-lp5n5   1/1     Running   0          4m2s   192.168.0.58   localhost.localdomain   <none>           <none>
my-nginx-86459cfc9f-t2zpp   1/1     Running   0          4m2s   192.168.0.59   localhost.localdomain   <none>           <none>
```

检查Pod的IP：

```bash
kubectl get pods -l run=my-nginx -o yaml | grep podIP
	podIP: 192.168.0.58
	podIP: 192.168.0.59
```

现在能够在集群中的任何节点并curl两个IP。注意，容器没有在节点上使用端口80，也没有任何特殊的NAT规则量流量路由到Pod。这意味着可以在同一个节点上运行多个nginx Pod，所有这些POd都是用相同的containerPort，并使用IP从集群中的任何其他Pod或节点访问它们。与Docker一样，端口仍然可以发布到主机节点的接口，但是由于网络模型的存在，对端口的需求已经大大减少。

