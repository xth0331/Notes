# 使用Keepalived进行初始负载均衡器配置



安装LoadBalancer软件包后，必须采取一些基本步骤来设置LVS路由器和用于Keepalived的真实服务器。

## 基本Keepalived配置

在基本配置示例中，两个系统配置负载均衡。LB1（活动）和LB2（备份）将路由请求四个Web服务器池，这些HTTP服务器使用真实的IP地址运行，IP为`192.168.1.20-192.168.1.24`，共享虚拟Ip地址`10.0.0.1`。每个负载均衡都有两个接口（`eth0和eth1`）一个用于处理外部流量，另一个用于将请求路由到真实服务器。使用的负载均衡算法是Round Robin，路由方法是Network Address Translation。

### 创建Keepalived.conf文件

Keepalived通过配置`keepalived.conf`为负载均衡器进行配置。要创建负载均衡，请在活动和备份负载均衡LB1和LB2中编辑`Keepalived.conf`文件。

#### 全局定义

全局定义部分允许管理员在发生负载均衡器更改时指定通知详细信息。全局配置定义是可选的。

```bash
global_defs {
    notification_email {
        admin@example.com
    }
    notification_email_from noreplay@example.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 60
}
```

`notification_email`是负载均衡器的管理员、`notification_email_from`是发送负载均衡器状态更改的地址、`SMTP`指邮件服务器。

#### VRRP实例

以下是LB1（主）中在`keepalived.conf`中`vrrp_sync_group`段的内容：

```bash
vrrp_sync_group VG1 {
    group {
        RH_EXT
        RH_INT
    }
}

vrrp_instance RH_EXT {
    state MASTET
    interface eth0
    virtual_route_id 50
    priority 100
    advert_int 1
    authentication {
        auth_type PAAS
        auth_pass passw123
    }
    virtual_ipaddress {
        10.0.0.1
    }
}

vrrp_instance RH_INT {
    state MASTER
    interface eth1
    virtual_route_id 2
    priority 100
    advert_int 1
    authentication {
        auth_type PAAS
        auth_pass passw123
    }
    virtual_ipaddress {
        192.168.1.1
    }
}
```

