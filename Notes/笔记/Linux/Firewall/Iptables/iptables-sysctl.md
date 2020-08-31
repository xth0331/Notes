# Iptables中有用的内核配置

## `rp_filter`

> 禁用路由三角剖分。响应查询出相同的接口，而不是另一个。还可以防止IP欺骗。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/conf/all/rp_filter = 1
EOF
```

## `log_martians`

> 启用记录格式错误的IP数据包的日志。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/conf/all/log_martians = 1
EOF
```

## `send_redirects`

> 禁止在所有接口上发送所有IPv4 ICMP重定向的数据包。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/conf/all/send_redirects = 0
EOF
```

## `accept_source_route`

> 禁用源路由数据包（设置了“严格源路由”或“松散源路由”选项的数据包）。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/conf/all/accept_source_route = 0
EOF
```

## `accept_redirects`

> 禁用接受ICMP重定向。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/conf/all/accept_redirects = 0
EOF
```

## `tcp_syncookies`

> 打开SYN泛洪保护（免受拒绝服务（DOS）攻击的保护）。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/tcp_syncookies = 1
EOF
```

## `icmp_echo_ignore_broadcasts`

> 禁用对Ping广播的响应。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/icmp_echo_ignore_broadcasts = 1
EOF
```

## `ip_forward`

> 启用IP路由。如果防火墙正在保护网络，包括NAT，则必须。

```bash
cat << EOF >> /etc/sysctl.d/40-custom.conf
net/ipv4/ip_forward = 1 
EOF
```

