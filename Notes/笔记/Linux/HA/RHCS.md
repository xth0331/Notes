# 部署RHCS

## 基础环境信息

描述本次基础环境信息：



### 系统信息

| 操作系统版本                                  | 操作系统镜像                     |
| --------------------------------------------- | -------------------------------- |
| `Red Hat Enterprise Linux Server release 6.8` | `rhel-server-6.8-x86_64-dvd.iso` |

### 主机信息

| 主机名       | 主机IP地址        | 角色            |
| ------------ | ----------------- | --------------- |
| `fence`      | x.x.x.x           | 仿真fence设备   |
| `RHCE-node1` | `192.168.220.221` | RHCS集群节点-01 |
| `RHEL-node2` | `192.168.220.222` | RHCS集群节点-02 |

### 存储信息

| 存储类型 | 多路径软件                |
| -------- | ------------------------- |
| `iSCSI`  | `device-mapper-multipath` |

**iSCSI只需要配置`initiator`与`target`可以请求即可，实际环境若为FC-SAN，请将主机的wwpn提供给存储管理员。**

**查看wwpn的方法**:`cat /sys/class/fc_host/host*/port_name`

##  环境准备

### 关闭防火墙

*暂时关闭，如果需要细化防火墙规则，文档最后会列出详细的防火墙配置*

```bash
iptables -F 		# 清空iptables规则（如果默认策略是Drop，请不要执行此条命令。）
iptables-save   # 保存配置
iptables -L -n    # 查看当前规则
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

   

