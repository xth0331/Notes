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
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE               NOMINATED NODE
my-nginx-65cd4c89f8-2ccbp   1/1     Running   0          2m43s   10.244.4.44   kubernetes-node5   <none>
my-nginx-65cd4c89f8-nnq7s   1/1     Running   0          2m43s   10.244.2.27   kubernetes-node3   <none>

```

检查Pod的IP：

```bash
kubectl get pods -l run=my-nginx -o yaml | grep podIP
	podIP: 10.244.4.44
	podIP: 10.244.2.27
```

现在能够在集群中的任何节点并curl两个IP。注意，容器没有在节点上使用端口80，也没有任何特殊的NAT规则量流量路由到Pod。这意味着可以在同一个节点上运行多个nginx Pod，所有这些POd都是用相同的containerPort，并使用IP从集群中的任何其他Pod或节点访问它们。与Docker一样，端口仍然可以发布到主机节点的接口，但是由于网络模型的存在，对端口的需求已经大大减少。

## 创建 Service

我们有Pod在一个扁平的、集群范围的地址空间中运行Nginx服务，可以直接连接到这些Pod，但如果某个节点死掉了会发生什么呢？Pod会终止，`Deployment`将创建新的Pod，且使用不同的IP。这正是`Service`要解决的问题。

Kubernetes Service从逻辑上定义了运行在集群中的一组Pod，这些Pod提供了了相同的功能。当每个`Service`创建时，会被分配一个唯一的IP地址（也成为`clusterIP`）。这个IP地址与一个`Service`的生命周期绑定在一起，当`Service`存在的时候它也不会改变。可以配置Pod使它与`Service`进行通信，Pod知道与`Service`通信将被自动地负载均衡到该`Service`中的某些Pod上。

可以使用`kubectl expose`命令为2个Nginx副本创建一个`Service`：

```bash
kubectl expose deployment/my-nginx
```

 这等价于使用`kubectl create -f` 命令创建，对应如下的yaml文件：

```yaml
appiVersion: v1
kind: Servvice
metadata: 
  name: my-nginx
  labels: 
    run: my-nginx
spec:
  ports:
  - port: 80 
    protocol: TCP
  selector:
    run: my-nginx
```

将创建一个`Sercice`,对应具有标签`run: my-nginx`的Pod，目标TCP端口80，并且在一个抽象的`Service`端口（targetPort：容器接收流量的端口；`port`: 抽象的Service端口，可以使任何其他Pod访问该Service端口）上暴露。

```bash
kubectl get svc my-nginx
NAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
my-nginx   10.0.162.149       <none>              80/TCP    21s
```

正如前面所提到的，一个`Service`由一组backend Pod组成。这些Pod通过`endpoints`暴露出来。`Service Selector`将持续评估，结果被POST到一个名称为`my-nginx`的EndPoint对象上。当Pod终止后，它会自动从Endpoint中移除，新的能够匹配上Service Selector的Pod将自动地被添加到Endpoint中。检查该Endpoint，注意到IP地址与在第一步创建的Pod是相同的。

```bash
kubectl describe svc my-nginx
Name:              my-nginx
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          run=my-nginx
Type:              ClusterIP
IP:                10.97.222.245
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.2.27:80,10.244.4.44:80
Session Affinity:  None
Events:            <none>
```

```bash
kubectl get ep my-nginx
NAME       ENDPOINTS                       AGE
my-nginx   10.244.2.27:80,10.244.4.44:80   13m
```

现在，能够从集群中任意节点上使用`curl`命令请求Nginx Service `<CLUSTER-IP>:<PORT>`。注意Service IP是完全是虚拟的，它从来没有走过网络。

## 访问Service

Kubernetes支持两种主要的服务 —— 环境变量和DNS 。 前者在单个节点上可以使用，然后后者必须使用`kube-dns`集群插件。

### 环境变量

 当Pod在Node上运行时，`kubectl`会为每个活跃的Service添加一组环境变量。这会有一个顺序的问题。想了解如何，检查正在运行的Nginx Pod的环境变量（Pod名称将不会相同）：

```bash
kubectl exec my-nginx 
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
```

