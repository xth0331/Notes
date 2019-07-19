# 部署RHCS

## 基础环境信息

描述本次基础环境信息：

![](https://blog-image.nos-eastchina1.126.net/9159.png)







### 系统信息

| 操作系统版本                                  | 操作系统镜像                     |
| --------------------------------------------- | -------------------------------- |
| `Red Hat Enterprise Linux Server release 6.8` | `rhel-server-6.8-x86_64-dvd.iso` |

### 

### 主机信息

| 主机名       | 主机IP地址        | 角色            |
| ------------ | ----------------- | --------------- |
| `fence`      | x.x.x.x           | 仿真fence设备   |
| `RHCE-node1` | `192.168.220.221` | RHCS集群节点-01 |
| `RHEL-node2` | `192.168.220.222` | RHCS集群节点-02 |

### RHCS中启用的IP端口

| IP 端口号  | 协议 | 组件                          |
| :--------- | :--- | :---------------------------- |
| 5404, 5405 | UDP  | `corosync/cman`（集群管理器） |
| 11111      | TCP  | `ricci`（推广更新的集群信息） |
| 21064      | TCP  | `dlm`（发布的锁定管理器）     |
| 16851      | TCP  | `modclusterd`                 |

### 存储信息

| 存储类型 | 多路径软件                |
| -------- | ------------------------- |
| `iSCSI`  | `device-mapper-multipath` |

**iSCSI只需要配置`initiator`与`target`可以请求即可，实际环境若为FC-SAN，请将主机的wwpn提供给存储管理员。**

**查看wwpn的方法**:`cat /sys/class/fc_host/host*/port_name`



## 注意事项

可使用各种方法配置红帽高可用性附加组件以满足您的需要。当进行计划、配置和实施的部署时，请考虑以下常规注意事项：

- 支持的集群节点数

  红帽高可用性附加组件最多支持的集群节点数为 16。

- 单点集群

  现在只能完全支持单点集群。

- GFS2

  虽然 GFS2 文件系统既可作为独立系统使用，也可作为集群配置的一部分，但不支持将 GFS2 作为单节点文件系统使用。红帽支持很多为单节点优化的高性能单节点文件系统，它们相对集群文件系统来说支出更低。红帽将继续为现有客户支持单节点 GFS2 文件系统。当您将 GFS2 文件系统作为集群文件系统配置时，必须确定该集群中的所有节点都可访问共享的文件系统。不支持不对称集群配置，在不对称集群中，有些节点可访问该文件系统，而其他节点则不能。这不要求所有节点确实挂载该 GFS2 文件系统。

- 无单点故障硬件配置

  集群可包括一个双控制器 RAID 阵列、多绑定链路、集群成员和存储间的多路径以及冗余无间断供电（UPS）系统以保证没有单点故障造成的应用程序失败或者数据丢失。另外，可设置一个低消耗集群以提供比无单点故障集群低的可用性。例如：您可以设置一个使用单控制器 RAID 阵列和只使用单以太网链路的集群。某些低消耗备选方案，比如主机 RAID 控制器、无集群支持的软件 RAID 以及多启动器平行 SCSI 配置与共享集群存储不兼容，或者不适合作为共享集群存储使用。

- 确保数据完整

  要保证数据完整，则每次只能有一个节点可运行集群服务和访问集群服务数据。在集群硬件配置中使用电源开关，就可让一个节点在故障切换过程中，重启节点 HA 服务前为另一个节点提供动力。这样就可防止两个节点同时访问同一数据并破坏数据。强烈建议使用 *Fence 设备*（远程供电、关闭和重启集群节点的硬件或者软件解决方案），以确保在所有失败情况下数据的完整性。

- 以太网通道绑定

  集群仲裁以及节点是否正常运行是由在通过以太网在集群节点间的沟通信息确定的。另外，集群节点使用以太网执行各种重要集群功能（例如：fencing）。使用以太网通道绑定，可将多个以太网接口配置为作为一个接口动作，这样就减小了在集群节点间以及其他集群硬件间典型切换的以太网连接单点故障风险。

- IPv4 和 IPv6

  高可用性附加组件支持 IPv4 和 IPv6 互联网协议。

##  环境准备

### 关闭防火墙

*暂时关闭，如果需要细化防火墙规则，文档最后会列出详细的防火墙配置*

```bash
iptables -F 		# 清空iptables规则（如果默认策略是Drop，请不要执行此条命令。）
iptables-save   # 保存配置
iptables -L -n    # 查看当前规则
```

### 禁用或删除NetworkManager

*不支持在集群节点中使用 `NetworkManager`。如果您已经在集群节点中安装了 `NetworkManager`，您应该删除或者禁用该程序。*

直接删除：

```bash
yum remove -y NetworkManager  # 移除包
```

或者不删除,考虑禁用：

```bash
service NetworkManager stop #关闭服务
chkconfig NetworkManager off # 取消开机启动
```

### 关闭SELinux

如果熟悉的话可以配置，不熟悉的话，就直接关闭吧。

```bash
setenforce 0
sed -i s/SELINUX=enforcing/SELINUX=disabled/g  /etc/selinux/config     
reboot
```

**彻底关闭SELinux需要重启，如果暂时不方便重启，可以先执行`setenforce 0`,等方便的时候重启服务器。**



### 配置本地Yum源

在`/etc/yum.repos.d`目录下创建新的本地`yumrepo`文件，本示例的`repo`文件名为`redhat-base.repo`。

```bash
mount /dev/sr0 /mnt
vi  /etc/yum.repos.d/redhat-base.repo
```

以下是文件内容，*(假设ISO文件挂载至`/mnt`下)*。

```bash
[base]
name=base
baseurl=file:///mnt
enabled=1
gpgcheck=0

[HighAvailability]
name=HighAvailablity
baseurl=file:///mnt/HighAvailability
enabled=1
gpgcheck=0

[ResilientStorage]
name=ResilientStorage
baseurl=file:///mnt/ResilientStorage
enabled=1
gpgcheck=0

[LoadBalancer]
name=LoadBalancer
baseurl=file:///mnt/LoadBalancer
enabled=1
gpgcheck=0
```

配置保存完成后，执行如下命令：

```bash
yum clean all 
yum makecache
```



### 添加存储

1. 安装`iscsi-initiator-utils`与`device-mapper-multipath`

   ```bash
   yum install -y iscsi-initiator-utils device-mapper-multipath
   ```

2. 发现`iscsi`target

   ```bash
   iscsiadm -m discovery -t sendtargets -p <targetIP1:PORT>	# 发现target
   iscsiadm -m discovery -t sendtargets -p <targetIP2:PORT> 	# 发现target
   chkconfig iscsi on   # 开机启动iscsi服务
   iscsiadm -m node -T <IQN1> -p <IP1:PORT> --login				# 登录
   iscsiadm -m node -T <IQN2> -p <IP2:PORT> --login				# 登录
   iscsiadm -m node -T <IQN1> -p <IP1:PORT> -op update -n node.startup -v automatic	# 自动登录
   iscsiadm -m node -T <IQN2> -p <IP2:PORT> -op update -n node.startup -v automatic	# 自动登录
   ```

3. 生成`/etc/multipath.conf`

   ```bash
   multipath --enable --with_multipathd y
   ```

4. 启动`multipathd`服务

   ```bash
   chkconfig multipathd on  			# 开机启动
   chkconfig --list multipathd 		# 检查
   service multipathd restart			# 重启服务
   service multipathd status 			# 检查
   ```

5. 查看多路径设

   ```bash
   multipath -ll  # 查看路径
   lsblk 				  # 查看设备
   ```



### 时间同步

ntp配置略



## 安装luci

1. 在节点1上安装`luci`

```bash
yum -y install luci
```

## 安装ricci,cman和rgmanager

1. 在两个RHCS节点上安装：

   ```bash
   yum -y install ricci cman rgmanager 
   ```

2. 配置`ricci`密码：

   ```bash
   passwd ricci
   ```



## 启动luci

使用`luci`配置集群要求在集群中安装并运行`ricci` 

1. 启动`ricci`服务：

     ```bash
      service ricci start  # 启动服务
      chkconfig ricci on # 开机启动
     ```

2. 启动`luci`服务：

   ```bash
   service luci start # 启动服务
   chkconfig luci on # 开机启动
   ```

3. 通过浏览器访问`https://LUCI-IP:8084`来访问luci，当启动luci服务时，登录`url`会回显到标准输出。

**注意**

初始情况下，只能通过root的身份验证信息访问；

如果 15 分钟后没有互动，则 **luci** 会处于闲置超时而让您退出。

 

## 创建集群

使用 **luci** 创建集群包括命名集群、在集群中添加集群节点、为每个节点输入 **ricci** 密码并提交创建集群请求。如果节点信息和密码正确，则 **Conga** 会自动在集群节点中安装软件（如果当前没有安装适当的软件包）并启动集群。按如下步骤创建集群：

1. 在 **luci** **「Homebase」**页面左侧菜单中点击**「管理集群」**。此时会出现**「集群」**页面。

   ![](https://blog-image.nos-eastchina1.126.net/luci-01.png)

   

2. 点击**「创建」**后出现**「创建集群页面」**。

   ![](https://blog-image.nos-eastchina1.126.net/luci-03.png)

   **图 3.3. 创建 luci 集群对话框**

3. 请根据需要在**「创建新集群」**页面中输入以下参数：

   - 在**「集群名称」**文本框中输入集群名称。集群名称不能超过 15 个字符。

   - 如果集群中的每个节点都有同样的 **ricci** 密码，您可以选择**「在所有节点中使用相同的密码」**，这样就可在您添加的节点中自动填写**「密码」**字段。

   - 在**「节点名称」**栏中输入集群中节点的名称，并在**「密码」**栏中为该节点输入 **ricci**密码。

   - 如果为您的系统配置了专门用于集群流量的专门的私有网络，则最好将 **luci** 配置为使用与集群节点名称解析拨通的地址与 **ricci** 进行沟通。您可以在**「Ricci 主机名」**中输入该地址达到此目的。

   - 如果您要在 **ricci** 代理中使用不同的端口，而不是默认的 11111 端口，您可以更改那个参数。

   - 点击**「添加另一个节点」**并输入节点名称，同时为集群的每个附加节点输入 **ricci** 密码。

   - 如果您不想要在创建集群时升级已经在节点中安装的集群软件软件包，请选择**「使用本地安装的软件包」**选项。如果您要升级所有集群软件软件包，请选择**「下载软件包」**选项。

     **注意**

     如果缺少任意基本集群组件（`cman`、`rgmanager`、`modcluster` 及其所有相依性软件包），无论是选择**「使用本地安装的软件包」**，还是**「下载软件包」**选项，都会安装它们。如果没有安装它们，则创建节点会失败。

   - 需要时选择**「加入集群前重启节点」**。

   - 如果需要集群的存储，则请选择**「启动共享存储支持」**。这样做将下载支持集群存储的软件包，并启用集群的 LVM。您应该只能在可访问弹性存储附加组件或者可扩展文件系统附加组件时选择这个选项。

4. 点击 **创建集群**。点击 **创建集群** 后会有以下动作：

   1. 如果您选择**「下载软件包」**，则会在节点中下载集群软件包。
   2. 在节点中安装集群软件（或者确认安装了正确的软件包）。
   3. 在集群的每个节点中更新并传推广群配置文件。
   4. 加入该集群的添加的节点

   显示的信息表示正在创建该集群。当集群准备好后，该显示会演示新创建集群的状态，请注意：如果没有在任何节点中运行 **ricci**，则该集群创建会失败。

   ![](https://blog-image.nos-eastchina1.126.net/luci-04.png)

5. 点击 **创建集群** 按钮创建集群后，您仍可以通过点击集群节点显示页面上部菜单中的**「添加」**或者**「删除」**功能从集群中添加或者删除节点。除非您要删除整个集群，否则必须在删除节点前停止它们。

   **注意**

   从集群中删除集群节点是一个破坏性操作，不能撤销。