# NSX Edge 安装



NSX Edge 为 NSX-T Data Center 部署外部的网络 NSX Edge提供路由服务和连接。如果要使用NAT、VPN等有状态服务部署Tier-0路由或Tier-1路由器，则需要部署NSX Edge由或Tier-1路由器，则需要部署NSX Edge。

> 每个NSX Edge节点只能具有一个Tier-0路由器。不过，可以在一个NSX Edge节点上托管多个Tier-1逻辑路由器。可以在同一集群中组合使用大小不同的NSX Edge 虚拟机，但不建议。



## Edge安装方案

> 从vSphere Web Client或命令行中通过OVA或OVF安装Edge时，在打开虚拟机电源前，不会验证属性值（用户名，密码或IP等）。

- 如果`admin`或`audit`用户指定用户名，名称必须是唯一的。如果指定了相同的名称，则会忽略该名称并使用默认名称（`admin`和`audit`）。
- 如果`admin`用户的密码不符合复杂性要求，必须通过SSH或控制台以`admin`用户身份（密码`default`）登录到NSX Edge。将提示更改密码。
- 如果`audit`用户的密码不符合复杂性要求，则会禁用该用户账户。要启用，请通过SSH或控制台以`admin`用户身份登录到NSX Edge，然后通过`set user audit`命令设置`audit`的密码。
- 如果`root`用户密码不符合复杂性要求，必须通过SSH或控制台以`root`身份登录（密码`vmware`)，将提示修改密码。

==只有设置足够复杂的密码后，设备的核心服务才会启动==

从OVA文件布署NSX Edge后，无法通过关闭虚拟机电源，然后从vCenter Server中修改OVA设置来更改虚拟机IP。

## NSX Edge 网络设置







