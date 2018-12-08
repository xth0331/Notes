# 使用HAPROXY和KEEPALIVED负载均衡简单示例

## 防火墙配置

```bash
firewall-cmd --zone=public --add-port 80/tcp --permanent
firewall-cmd --reload
```

指定自己需要开放的端口来替换80/tcp

## 安装配置Keepalived



1. 两个节点安装Keepalived

   ```bash
   yum -y install keepalived
   ```

2. 编辑Keepalived配置文件

   ```bash
   vim /etc/keepalive/keepalived.conf
   ```

   内容如下：

   ```bash
   vrrp_script chk_haproxy {
     script "killall -0 haproxy" # check the haproxy process
     interval 2 # every 2 seconds
     weight 2 # add 2 points if OK
   }
   
   vrrp_instance VI_1 {
     interface eth0 # interface to monitor
     state MASTER # MASTER on haproxy, BACKUP on haproxy2
     virtual_router_id 51
     priority 101 # 101 on haproxy, 100 on haproxy2
     virtual_ipaddress {
       192.168.0.100 # virtual ip address
     }
     track_script {
       chk_haproxy
     }
   }
   ```

3. 启动服务：

   ```bash
   systemctl enable keepalived ; systemctl start keepalived
   ```




## 安装配置HAProxy

1. 安装HAProxy。

   ```bash
   yum -y install haproxy
   ```

2. 配置HAProxy。

   ```bash
   vim /etc/haproxy/haproxy.cfg
   ```

   内容：

   ```bash
   frontend http_web *:80
       mode http
       default_backend rgw
   
   backend rgw
       balance roundrobin
       mode http
       server  rgw1 10.0.0.71:80 check
       server  rgw2 10.0.0.80:80 check
   ```

3. 启动服务：

   ```bash
   systemctl enable haproxy ; systemctl start haproxy
   ```
