# 监控和诊断性能问题（CPU）

以下工具对于处理器机器配置相关的系统性能和监控性能问题诊断有很大帮助。



## turbostat

`turbostat`在规定的间隔中给出计时器的结果以协助识别服务器异常，例如过度耗电，无法进入睡眠状态或是创建了不必要的系统管理中断（SMIs）。





## numastat

`numastat`工具会列举出每个NUMA节点内存数据给所有的进程和操作系统，并会告知进程内存是散布于系统还是集中于某个节点。

通过处理器的`top`输出进行交互参照`numastat`输出，已确认进程线程是在同一个节点运行，此节点是进程内存分配节点。



## /proc/interrupts

`/proc/interrupts`文件列举了从一个特殊的I/O设备发送至各处理器的中断数量，显示了中断请求(IRQ)数量、系统中个处理器该类型中断请求的数量，发送的中断类型以及都好分隔开的回应所列出中断请求的设备列表。

如果一个特定的应用程序或是设备生成大量的中断请求给远程处理器，其性能就会收到影响。这种情况下，当应用程序或设备在处理中断请求时，可以在同一节点设置一个处理器，缓解性能不佳的情况。



## 使用pqos监控缓存和内存带宽

`pqos`实用程序，可从`intel-cmt-cat`包获得，使您可以监控CPU缓存和内存带宽最近Intel处理器。

`pqos`工具提供了类似于高速缓存和内存监测工具`top`工具。它监测：

- 每个周期的指令（IPC）。
- 最后一级缓存MISSES的计数。
- 在给定CPU中执行的程序在LLC中占用的大小（以KB为单位）。
- 本地内存的带宽（MBL）。
- 远程内存的带宽（MBR）。

使用以下命令启动监视工具：

```bash
pqos --mon-top
```

输出中的项目按最高LLC占用率排序。