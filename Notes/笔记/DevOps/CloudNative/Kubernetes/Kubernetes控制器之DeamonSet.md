# Kubernetes控制器之DeamonSet

一个`DeamonSet`确保所有（或部分）节点上运行Pod的副本。随着节点添加到集群，将添加Pod。随着节点从集群中删除，这些Pod将被垃圾回收。删除`DeamonSet`将清除它创建的Pod。

`DeamonSet`的一些典型用法：

- 运行集群存储后台守护进程，例如每个节点上的`glusterd`,`ceph`。
- 在每个节点上运行日志收集守护进程，例如`fluentd`或`logstash`。
- 在每个节点上运行节点监控守护进程，例如`Prometheus Node Exporter`，`collectd`，`Dynatrace`，`OneAgent`，`APPDynamics Agent`, `Datadog agent`， `New Relic agetn`， `Ganglia` gmond, 或Instana agent.



在一个简单的例子中，覆盖所有节点的一个`DeamonSet`将用于每种类型的守护进程。更复杂的设置可能会为单一类型的守护进程使用多个`DeamonSet`，但对不同的硬件类型使用不同的标志和/或不同的内存和CPU请求。

## 编写 DaemonSet Spec

### 创建 DaemonSet

可以在`YAML`文件中描述`DeamonSet`，