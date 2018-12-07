# Keepalived直接路由配置

Keepalived的直接路由配置与NAT配置类似。在以下示例中，Keepalived配置为在80端口上为一组运行HTTP的真实服务器提供负载均衡。要配置直接路由，将`lb_kind`参数更改为`DR`。

以下示例显示使用直接路由的Keepalived配置活动服务器的`keepalived.conf`配置文件。

```bash
global_defs {
    notification_email {
      admin@example.com
    }
    notification_email_from noreply_admin@example.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 60   
}

vrrp_instance RH_1 {
    state MASTER
    interface eth0
    virtual_router_id 50
    priority 100
    advert_int 1
    authentication {
        auth_type PAAS
        auth_paas paasw123
    }
    virtual_ipaddress {
        172.32.0.1
    }
}

virtual_server 172.31.0.1 80
	delay_loop 10
	lb_algo rr
	lb_kind DR
	persistence_timeout 9600
	protocol TCP
	
	real_srrver 192.168.0.1 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 10
          connect_port	80
        }
	}
	real_server 192.168.0.2 80 {
        werght 1
        TCP_CHECK {
          connect_timeout 10
          connect_port	80
        }
	}
	real_server 192,168.0.3 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 10
          connect_port	80
        }
	}
}
```

以下示例显示直接路由的Keepalived配置中备份服务器的`keepalived.conf`文件。状态和优先级值与活动服务器不同。

```bash
global_defs {
   notification_email {
     admin@example.com
   }
   notification_email_from noreply_admin@example.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 60
}

vrrp_instance RH_1 {
    state BACKUP
    interface eth0
    virtual_router_id 50
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass passw123
    }
    virtual_ipaddress {
        172.31.0.1
    }
}

virtual_server 172.31.0.1 80
    delay_loop 10
    lb_algo rr
    lb_kind DR
    persistence_timeout 9600
    protocol TCP

    real_server 192.168.0.1 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 10
          connect_port    80
        }
    }
    real_server 192.168.0.2 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 10
          connect_port    80
        }
    }
    real_server 192.168.0.3 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 10
          connect_port    80
        }
    }
}
```

