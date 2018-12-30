# Tuna

可以使用`tuna`程序调整调度程序可调参数，调整线程优先级，`IRQ`处理程序以及隔离CPU核心和套接字，`tuna`旨在降低执行调优任务的复杂性。



## 用tuna查看系统

可以使用`tuna`显示系统当前正在发生的情况。

查看当前的策略和优先事项，使用`tuna --show_threads`命令：

```bash
tuna --show_threads
                      线程       ctxt_switches
  pid   SCHED_   rtpri affinity voluntary nonvoluntary           cmd
  1      OTHER     0      0,1      1741         1927         systemd
  2      OTHER     0      0,1       162            0        kthreadd
  3      OTHER     0        0      1372            0     ksoftirqd/0
  5      OTHER     0        0         8            0    kworker/0:0H
  6      OTHER     0      0,1       136            0  kworker/u256:0
  7       FIFO    99        0       329            0     migration/0
  8      OTHER     0      0,1         2            0          rcu_bh
  9      OTHER     0      0,1     21314            0       rcu_sched
  10      FIFO    99        0      2132            0      watchdog/0
  11      FIFO    99        1      2132            0      watchdog/1
  12      FIFO    99        1       279            0     migration/1
  13     OTHER     0        1       786            0     ksoftirqd/1
  15     OTHER     0        1        11            0    kworker/1:0H
  17     OTHER     0      0,1       162            0       kdevtmpfs
  18     OTHER     0      0,1         2            0           netns
  19     OTHER     0      0,1        72            0      khungtaskd
  20     OTHER     0      0,1         2            0       writeback
```



想要仅显示与PID对应的特定线程或匹配命令名称，在`--show_threads`之前添加`--threads`:

```bash
tuna --threads=PID_OR_CMD_LIST --show_threads
```

查看当前中断请求（IRQ）及其相关性，使用`tuna --show_irqs`命令：

```bash
tuna --show_irqs
```

仅显示与IRQ编号对应的特定中断请求或匹配IRQ用户名，在`--show_irqs`之前添加`--irqs`选项：

```bash
tuna --irqs=NUMBER_OR_USER_LIST --show_irqs
```

## 使用tuna调整CPU

`tuna`命令可以针对单个CPU，要列出系统上的CPU，`/proc/cpuinfo`文件可以获取详细信息。

要指定受命令影响的CPU列表，使用：

```bash
tuna --cpus=CPU_LIST --COMMAND
```

隔离CPU会导致当前该CPU上运行的所有任务移动到下一个可用的CPU。要隔离CPU，使用：

```bash
tuna --cpus=CPU_LIST --isolate
```

包含CPU允许线程在指定的CPU上运行。要包含CPU，运行：

```bash
tuna --cpu=CPU_LIST --include
```

`CPU_LIST`参数是逗号分隔的CPU编号列表。例如，`--cpus=0,2`。

## 使用tuna调整IRQS

要查看系统上当前运行的IRQ列表，查看`/proc/interrpupts`文件。也可以使用`tuna --show_irqs`命令。

要指定受命令影响的IRQ列表，使用`–irqs`参数：

```bash
tuna --irqs=IRQ_LIST --COMMAND
```

要将终端移动到指定的CPU，请使用`--move`参数：

```bash
tuna --irqs=IRQ_LIST --cpus=CPU_LIST --move
```

`IRQ_LIST`参数是逗号分隔的`IRQ`编号或用户名模式的列表。

`CPU_LIST`参数是逗号分隔的CPU编号列表。例如，`--cpus=0,2`。

例如，要定位名称以`sfc1`开头的所有的中断，并将他们分布在两个CPU：

```bash
tuna --irqs=sfc1\* --cpus=7,8 --move --spread
```

要验证设置，使用`--move`参数修改`IRQ`之前和之后使用`-show_irqs`参数：

```bash
tuna --irqs=128 --show_irqs
tuna --irqs=128 --cpus=3 --move
tuna --irqs=128 --show_irqs
```

可以比较出更改前后`IRQ`的状态。

## 使用tuna调整任务

要更改线程的策略和优先级信息，使用`--priority`参数：

```bash
tuna --rhreads=PID_OR_CMD_LIST --priority=[POLICY:]RT_PRIORITY
```

- *PID_OR_CMD_LIST*是逗号分隔的PID或命令名模式的列表。
- 对于默认策略，将策略设置为**RR**（roind-robin），**FIFO**（first in ， first out）或者**OTHER**。
- 将RT_PRIORITY设置为范围1-99。1是最低优先级，99是最高优先级。



例如：

```bash
tuna --threads=7861 --priority=RR:40
```

要验证设置的更改，请在修改`--proprity`之前和之后使用`--show_threads`参数查看：

```bash
tuna --rhreads=sshd --show_threads --priority=RR:40 --show_threads
```

这允许比较更改之前和之后所选线程的状态。

## tuna配置示例

**将任务分配给特定CPU**

以下示例使用具有四个或更多处理器的系统，并显示如何使所有`ssh`线程在CPU 0和1以及`http`CPU 2和3上的所有线程上运行。

```bash
tuna --cpus=0,1 --threads=ssh\* --move --cpus=2,3 --threads=http\* --move
```

上面的示例命令按顺序执行以下操作：

1. Select CPUs 0 and 1.
2. Select all threads that begin with `ssh`.
3. Move the selected threads to the selected CPUs. Tuna sets the affinity mask of threads starting with `ssh` to the appropriate CPUs. The CPUs can be expressed numerically as 0 and 1, in hex mask as `0x3`, or in binary as `11`.
4. Reset the CPU list to 2 and 3.
5. Select all threads that begin with `http`.
6. Move the selected threads to the selected CPUs. Tuna sets the affinity mask of threads starting with `http` to the appropriate CPUs. The CPUs can be expressed numerically as 2 and 3, in hex mask as `0xC`, or in binary as `1100`.

**查看当前配置**

以下示例使用`--show_threads`（`-P`）参数显示当前配置，然后测试所请求的更改是否按预期进行。

```bash
tuna --threads=gnome-sc\* \
        --show_threads \
        --cpus=0 \
        --move \
        --show_threads \
        --cpus=1 \
        --move \
        --show_threads \
        --cpus=+0 \
        --move \
        --show_threads

                       thread       ctxt_switches
     pid SCHED_ rtpri affinity voluntary nonvoluntary             cmd
   3861   OTHER     0      0,1     33997           58 gnome-screensav
                       thread       ctxt_switches
     pid SCHED_ rtpri affinity voluntary nonvoluntary             cmd
   3861   OTHER     0        0     33997           58 gnome-screensav
                       thread       ctxt_switches
     pid SCHED_ rtpri affinity voluntary nonvoluntary             cmd
   3861   OTHER     0        1     33997           58 gnome-screensav
                       thread       ctxt_switches
     pid SCHED_ rtpri affinity voluntary nonvoluntary             cmd
   3861   OTHER     0      0,1     33997           58 gnome-screensav
```

The above example command performs the following operations sequentially:

1. Select all threads that begin with `gnome-sc`.
2. Show the selected threads to enable the user to verify their affinity mask and RT priority.
3. Select CPU 0.
4. Move the `gnome-sc` threads to the selected CPU (CPU 0).
5. Show the result of the move.
6. Reset the CPU list to CPU 1.
7. Move the `gnome-sc` threads to the selected CPU (CPU 1).
8. Show the result of the move.
9. Add CPU 0 to the CPU list.
10. Move the `gnome-sc` threads to the selected CPUs (CPUs 0 and 1).
11. Show the result of the move.