# NSX Manager安装

NSX Manager提供`GUI`和`REST API`以创建、配置和监控NSX-T Data Center组件，例如，逻辑交换机、逻辑路由器和防火墙。

NSX Manager提供了系统视图并且是NSX-T Data Center的管理组件。

为了获得高可用性，NSX-T Data Center支持三个NSX Manager的管理集群。对于生产环境，建议部署管理集群。

在vSphere环境中，NSX Manager支持以下功能：

- vCenter Server 可以使用vMotion功能在主机和集群之间实时迁移NSX Manager。
- vCenter Server 可以使用Storage vMotion功能在主机和集群之间实时迁移NSX Manager。
- vCenter Server 可以使用DRS功能在主机和集群之间重新平衡NSX Manager。
- vCenter Server 可以使用反关联性功能在主机和集群之间管理NSX Manager。

==NSX Manager 导入过程略==



## NSX Manager 单站点要求和建议

- 建议将NSX Manager放置在不同的主机上，避免一个主机故障影响多个管理器。
- 各NSX之间最大延迟为10毫秒。
- 可以将NSX Manager放置在不同的vSphere集群或一个共同的集群中。
- 建议将NSX Manager放置在不同的管理子网一个共享的管理子网中。在使用vSphere HA时，建议使用一个共享管理子网，这样由vSphere恢复的NSX Manager便可以保留其IP。
- 此外，还建议将NSX Manager放置在共享存储上。

## 登录NSX Manager

安装完成后，可以使用GUI执行其他安装任务。

1. 从浏览器登录`https://<nsx-manager-ip-address>`中的NSX Manager。
2. 阅读并接受EULA。
3. 选择是否加入CEIP。
4. 保存。



