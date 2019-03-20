# Kubernetes控制器之StatefulSet

`StatefulSet`是用于管理有状态应用程序工作负载的API对象。

管理一组Pod的部署和扩展，并提供有关这些Pod的排序和唯一性保证。

和`Deployment`类似，`StatefulSet`管理基于相同容器规范的Pod。与`Deployment`，`StatefulSet`为其每个Pod维护了一个粘性标识。这些Pod是根据相同的规范创建的，但不可互换：每个Pod都有一个持久的标识符，它在任何重新安排时都会保留。

`StatefulSet`以与任何其他Controller相同的模式运行。在`StatefuleSet`对象中定义所需的状态，`StatefuleSet`控制器进行任何必要的更新已从当前状态到达那里。

## 使用`StatefuleSet`

`StatefuleSet`对于需要以下一项或多项的应用程序非常有用。