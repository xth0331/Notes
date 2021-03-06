

# HAProxy 调度算法

可以在`/etc/haproxy/haproxy.cfg`配置文件的`backend`部分的`balance`参数中编辑用于负载均衡的HAProxy调度算法。注意，HAProxy支持具有多个后端的配置，并且每个后端都可以配置调度算法。

**Round-Robin(roundrobin)**

按顺序在真实服务器池分配每个请求。使用此算法，所有真实服务器都被视为同等，而不考虑容量或负载。此调度模型类似于循环DNS，但由于它是基于网络连接而不是主机，因此更加精细。负载均衡循环调度也不会遭受DNS缓存查询导致的不平衡。但是在HAProxy中，由于服务器的权重配置可以使用此调度程序即时完成，因此活动服务器的数量限制为每个后端。

**静态循环（static-rr）**

像Round-Robin一样，围绕真实服务器池顺序分配每个请求，但不允许动态配置权重。但是，由于服务器权重的静态特性，后端的活动服务器数量没有限制。

**最少连接（leastconn）**

将更多请求分发给具有较少活动连接的真实服务器。具有不同会话或连接长度的动态环境的管理员可能会发现此调度程序更适合其环境。它也是一组服务器具有不同容量的环境的理想选择，因为管理员可以使用此调度程序即时调整权重。

**来源(source)**

通过散列请求源IP地址并除以所有正在运行的服务器的权重来分配对服务器的请求，以确定那个服务器将获得请求。在所有服务器都在运行的情况下，源ip请求将始终由同一个真实服务器提供服务。如果正在运行的服务器的数量或容量发生变化，则会话可能移动到另一个服务器，因为散列/权重结果已更改。

**URI（uri）**

通过散列整个URI（或URI的可配置部分）将请求分发给服务器，并除以所有正在运行的服务器的权重，以确定请求的服务器。在所有活动服务器都在运行的情况下，目标IP请求将始终由统一真实服务器提供服务。可以通过URI的目录部分的开头处的字符长度来进一步配置该调度器，以计算散列结果和URI中的目录深度（由URI中的正斜杠指定）以计算散列结果。

**网址参数（url_param）**

通过在源URL请求中查找特定参数字符串并执行哈希计算除以所有正在运行的服务器的权重，将请求分发到服务器。如果URL中缺少该参数，则调度程序默认为循环调度。可以基于POST参数以及基于管理员在计算散列结果之前为特定参数的权重分配的最大八位字节的数量来使用修饰符。

**标题名称（hdr）**

通过检查每个源HTTP请求中的特定标头名称并执行散列计算除以所有正在运行的服务器的权重，将请求分发到服务器。如果标头不存在，则调度程序默认为循环调度。

**RDP Cookie (rdp-cookie)**

通过查找每个TCP请求的RDP cookie并执行哈希计算除以所有正在运行的服务器的权重，将请求分发给服务器。如果标头不存在，则调度程序默认为循环调度。此方法非常适合持久性，因为可以保持会话完整性。

 