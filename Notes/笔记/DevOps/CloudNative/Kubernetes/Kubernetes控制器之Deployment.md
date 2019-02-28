# Kubernetes控制器之Deployment

`Deployment controller`为Pod和`ReplicaSet`提供声明式更新。

在`Deployment`对象中描述了所需的状态，`Deployment`控制器将实际状态更改为所需状态。您可以定义`Deployment`以创建`ReplicaSet`，或者删除现有的部署并使用新的`Deployment`采用所有资源。

> 不应该管理`Deployment`所拥有的`ReplicaSet`。



## 创建`Deployment`

以下是`Deployment`的示例。它创建了一个`ReplicaSet`来产生三个`nginx`Pod：

**nginx-deployment.yaml：**

```yaml
apiVerison: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        - containerPort: 80
```

