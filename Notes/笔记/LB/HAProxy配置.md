# HAProxy配置

## 全局配置

`global`设置配置适用于运行HAProxy的所有服务器的参数。`global`部分可能如下：

```bash
global
	log 127.0.0.1 local2
	maxconn 4000
	user haproxy
	group haproxy
	daemon
```

在上面的配置中，管理员已将服务配置`log`为本地`syslog`服务器的所有条目。默认情况下，这可能是`/var/log/syslog`或某些指定的位置。

`maxconn`参数指定服务器的最大并发连接数。默认情况下，最大值为2000。

`user`和`group`参数指定`haproxy`进程所属的用户名和租名。

最后`daemon`参数指定`haproxy`作为后台进程运行。

## 默认设置

`default`设置配置适用于所有代理子节的配置参数（`frontend`,`backend`,`listen`）。配置如下所示：

> proxy 子节（*frontend*，*backend*，*listen*）中配置任何参数都优先于`default`中的参数值。

```bash
defaults
	mode					http
	log						global
	option					httplog
	option					dontlognull
	retries					3
	timeout http-request	10s
	timeout queue			1m
	timeout connect			10s
	timeout client			1m
	timeout server			1m
```

`mode`指定HAProxy实例的协议。使用`http`模式将源请求连接到基于HTTP的真实服务器，非常适合Web服务器负载均衡。对于其他程序，请使用`tcp`模式。

`log`指定日志条目的日志地址和`syslog`工具。HAProxy将全局配置中的log参数引用。

`option httplog`允许记录HTTP会话，包括HTTP请求，会话状态，连接号，原地址和连接计数器以及其他值。

`option httplognull`禁用空连接的记录，这意味着HAProxy不会记录没有传输数据的连接。建议不要讲此类环境用于Internet上的web应用程序，其中空连接可能表示恶意活动，例如漏洞的开放端口扫描。`retries`指定真实服务器在第一次尝试连接失败后重试连接情趣的次数。

`timeout`指定请求连接或相应的不活动时间长度。这些值通常以毫秒为单位，但可以通过以任何单位表示，支持的单位是`us`（微秒），`ms`（毫秒），`s`（秒），`m`（分钟），`h`（小时），`d`（天）。

`http-request 10s`等待来自客户端的完整HTTP请求10秒。`quue 1m`将一分钟设置为连接断开之前等待的时间，并且客户端收到`503`错误。`connect 10s`指定等待成功连接到服务器的秒数。`client 1m`指定客户单可以保持非活动状态的时间量。`server 1m`指定服务器在超时发生之前接受或发送数据的时间。

## 前端设置

这些`frontend`设置为服务器的监听套接字配置客户端连接请求。HAProxy配置`frontend`如下：

```bash
frontend main
  bind 192.168.0.10:80
  default_backend app
```

在`frontend`的`main`配置为`192.168.0.10`的IP地址，并使用`bind`参数指定监听80端口。连接后，使用`backend`指定所有会话连接到`app`后端。

## 后端设置

在`backend`设置中指定真实服务器的IP地址以及负载均衡调度算法。`backend`部分如下：

```bash
backend app
	balance		roundrobin
	server	app1	192.168.1.1:80 check
	server  app2	192.168.1.2:80 check
	server  app3    192.168.1.3:80 check inter 2s rise 4 fall 3
	server 	app4	192.168.1.4:80 backup
```

后端服务器已命名*app*。在*balance*指定要使用的负载均衡的调度算法，在这种情况下是循环（*roundrobin*），但也可以是通过HAProxy的支持的任何调度。

这些*server*行指定后端可用的服务器。*app1*to *app4*是每个真实服务器内部分配的名称。日志文件将按名称指定服务器消息。地址是分配的IP地址。IP地址中冒号后面的值是特定服务器上连接发生的端口号。该*check*选项标记服务器以进行定期运行状况检查，以确保它可用并能够接收和发送数据并获取会话请求。服务器app3还将运行状况检查间隔配置为两秒（*inter 2s*），app3必须通过的检查量以确定服务器是否被视为正常（*rise 4*），以及服务器在被视为失败之前连续未通过检查的次数（*fall 3*） 。