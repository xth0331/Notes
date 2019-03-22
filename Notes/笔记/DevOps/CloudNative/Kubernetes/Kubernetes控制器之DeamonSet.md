# Kubernetes控制器之DeamonSet

一个`DeamonSet`确保所有（或部分）节点上运行Pod的副本。随着节点添加到集群，将添加Pod。随着节点从集群中删除，这些Pod将被垃圾回收。删除`DeamonSet`将清除它创建的Pod。

