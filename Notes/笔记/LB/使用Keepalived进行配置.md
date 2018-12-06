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

LB2（备份路由）`keepalived.conf`中`vrrp_sync_group`段配置：

```bash
vrrp_sync_group VG1 {
    group {
        RH_EXT
        RH_INT
    }
}

vrrp_instance RH_EXT {
    state BACKUP
    interface eth0
    virtual_router_id 50
    priority 99
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



在这些示例中，`vrrp_sync_group`定义了通过任何状态更改（例如故障转移）保持在一起的VRRP组。为外部接口定义了一个与Internet通信（RH_EXT），以及用于内部接口（RH_INT）的实例。

`vrrp_instance`详细说明了VRRP服务守护进程的虚拟接口配置，守护进程创建虚拟IP实例。`state MASTER`指定活动服务器，`state BACKUP`指定备份服务器。

`interface`参数将物理接口名称分配给此特定虚拟IP实例。

`virtual_route_id`是虚拟路由器实例的数字标识符。在参与此虚拟路由器的所有LVS路由器系统上必须相同。它用于区分`keepalived`在同一网络接口上运行的多个实例。

`priority`所述分配分配界面接管在故障转移的顺序；数字越大，优先级越高。此优先级值在0到255的范围内，并且`state`配置为`MASTER`的优先级应设置为高于`state`配置为`BACKUP`的服务器 。

`authentication`指定于对服务器进行身份验证以进行故障转移同步身份验证类型（`auth_type`）和密码(`auth_pass`)。`PAAS`指定密码认证；Keepalived还支持`AH`或验证标头以及实现连接完整性。

最后，该`virtual_ipaddress`选项指定接口虚拟IP地址。

#### 虚拟服务器定义

LB1和LB2上的`keepalived.conf`中文件的虚拟服务器定义部分相同。

```bash
virtual_server 10.0.0.1 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    protocol TCP

    real_server 192.168.1.20 80 {
        TCP_CHECK {
                connect_timeout 10
        }
    }
    real_server 192.168.1.21 80 {
        TCP_CHECK {
                connect_timeout 10
        }
    }
    real_server 192.168.1.22 80 {
        TCP_CHECK {
                connect_timeout 10
        }
    }
    real_server 192.168.1.23 80 {
        TCP_CHECK {
                connect_timeout 10
        }
    }

}
```



在此块中，`virtual_server`首先使用IP地址配置。然后`delay_loop`配置运行状况检查之间的时间量（以秒为单位）。`lb_algo`选项指定用于可用性的算法类型（在本例中，`rr`对于Round-Robin。`lb_kind`选项确定路由方法，在这种情况下使用网络地址转换（或`nat`）。

配置虚拟服务器详细信息后`real_server`，再次通过首先指定IP地址来配置选项。该`TCP_CHECK`节使用TCP检查真实服务器的可用性。`connect_timeout`配置发生超时之前的秒数。

| 算法名称                         | `lv_algo` |
| -------------------------------- | --------- |
| 轮循                             | `rr`      |
| 加权循环法                       | `wrr`     |
| 最小连接                         | `lc`      |
| 加权最小连接                     | `wlc`     |
| 基于位置的最小连接               | `lblc`    |
| 具有复制的基于位置的最小连接调度 | `lblcr`   |
| 目的地哈希                       | `dh`      |
| 来源哈希                         | `sh`      |
| 来源预期延迟                     | `sed`     |
| 从不排队                         | `nq`      |