# Tuned

Tuned是一个守护进程，用于`udev`监视连接的设备，并根据所选的配置文件静态和动态地调整系统设置。Tuned分布有许多预定义的配置文件，适用于高吞吐量，低延迟或`powersave`等常见用例。可以修改为每个配置文件定义的规则，并自定义如何调整特定设备。要还原特定配置文件对系统设置所做的所有更改，可以切换到另一个配置文件或停止`tuned`服务。

> Tuned 支持 **Tuned in on-deamon mode**，此模式下，`tuned`会应用设置并退出。要启用这种模式，在`/etc/tuned/tuned-main.conf`文件中设置`deamon = 0` 。

静态调优主要包括预定义的`sysctl`和`sysfs`设置以及一些配置工具的一次性激活。`tuned`还根据监控信息动态监控系统组件的使用并调整系统设置。

动态调整考虑了在任何给定系统的整个正常运行时间内各种系统组件的使用方式不同。例如，阴干驱动器在启动和登录期间大量使用，但稍后再用户可能主要使用web浏览器或电子邮件客户端几乎不适用，类似地，CPU和网络设备在不同时间使用特点也不同。`tuned`监控这些组件的活动并对其使用的变化做出反应。

典型的，办公室工作站。大多数情况下，以太网网络接口不是很活跃。每隔一段时间只有少量的电子邮件进出，或者可能会加载一些网页。对于那些类型的负载，网络接口不必全速运行，就像默认情况下那样。`tuned`有一个用于网络设备的监控和调整插件，可以检测到这活动，然后自动降低网络速率，通常为了降低功耗。如果界面上的活动增减了很长一段时间。例如正在下载,`tuned`检测到这一点并将接口速率设置为最大，以便活动级别如此之高时提供最佳性能。此原则也用于CPU和硬盘的其他插件。

动态`tuned`默认是禁用的，可以更改`/etc/tuned/tuned-main.conf`文件中的`dynamic_tuning`为`1`来启用。

## 插件

`tuned`使用两种类型的插件：*tuning plugins*和,*monitoring plugins*,目前有一下monitoring plugins：

- disk

  获取每个设备的磁盘负载（IO操作数）和测量间隔。

- net

  获取每个网卡的网络负载（传输的数据包数）和测量间隔。

- load

  获取每个CPU的CPU负载和测量间隔。

可能通*tuning plugins*进行动态调整来使用*monitoring plugins*的输出。当前实现的动态调整算法尝试平衡性能和`powersave`,因此在性能配置文件中被禁用（可以在`tuned`配置文件中启用或禁用各个插件的动态调整）。只要任何启用的调优插件需要其指标，*monitoring plugins*就会自动实例化，如果两个*tuning plugins*需要相同的数据，则只创建一个*monitoring plugins*实例并共享数据。

每个*tuning plugins*都会调整单个子系统，并从**调整后的**配置文件中获取几个参数。每个子系统可以具有多个设备（例如，多个CPU或网卡），这些设备由*tuning plugins*的各个实例处理。还支持各个设备的特定设置。提供的配置文件使用通配符来匹配各个子系统的所有设备，这允许插件根据所需膜表调整这些子系统（选定的配置文件）并且用户唯一需要做的就是选择正确的`tuned`配置文件。

目前，实现了以下*tuning plugins*：

**cpu**

​	将CPU调度器设置为`governor`参数指定的值，并根据CPU负载动态更改PM Qos CPU DMA延迟。如果CPU负载低于*load_threshold*参数指定的值，则将latency设置为*latency_high*参数指定的值，否则将其设置为*latency_low*指定的值。侧歪，可以将延迟强制为特定值，而无需进一步动态更改。这可以通过将*force_latency*参数设置为所需的延迟值来实现。



**eeepc_she**

​	根据CPU负载情况动态设置FSB速度；一般不需要配置，如果CPU负载低于或等于`load_threshold_powersave`参数指定的值，则插件将FSB速度设置为参数指定的值。

**net**

​	将`wake-on-lan`配置为`wake_on_lan`参数指定的值。还根据接口利用率动态改变接口速率。

**sysctl**

​	设置`sysctl`插件参数指定的各种设置。语法为`name = value`，其中`name`与`sysctl`工具提供的名称相同。如果需要更改其他插件未涵盖的设置，请使用此插件。

**usb**

​	将USB设备的自动暂停超时设置为`autosuspend`参数指定的值。0表示禁用自动挂起。

**audio**

​	将音频解码器的自动暂停超时设置为`timeout`参数指定的值。目前`snd_hda_intel`和`snd_ac97_aodec`得到支持。该值为0表示禁用自动暂停。

**disk**

将电梯设置为*elevator*参数指定的值。它还将ALPM设置为参数指定的值*alpm*，ASPM为参数指定的值*aspm*，scheduler quantum为*scheduler_quantum*参数指定的值，disk spindown timeout为*spindown*参数指定的值，disk readahead为指定的值*readahead*参数，可以将当前磁盘预读值乘以*readahead_multiply*参数指定的常量。此外，此插件根据当前驱动器利用率动态更改驱动器的高级电源管理和spindown超时设置。动态调整可以通过布尔参数控制*dynamic*，默认情况下启用。

**mounts**

根据*disable_barriers*参数的布尔值启用或禁用装载障碍。

**script**

此插件可用于执行在加载或卸载配置文件时运行的外部脚本。该脚本由一个参数调用，该参数可以是`start`或`stop`（它取决于在配置文件加载或卸载期间是否调用脚本）。脚本文件名可以由*script*参数指定。请注意，您需要在脚本中正确实现停止操作并恢复在启动操作期间更改的所有设置，否则回滚将不起作用。为了您的方便，`functions`默认情况下安装Bash帮助程序脚本，允许您导入和使用其中定义的各种函数。请注意，此功能主要是为了向后兼容性而提供的，建议您将其用作最后的手段，如果它们涵盖了所需的设置，则更喜欢其他插件。

**sysfs**

设置`sysfs`插件参数指定的各种设置。语法是`name`= `value`，其中`name`是`sysfs`要使用的路径。如果您需要更改其他插件未涵盖的某些设置，请使用此插件（如果它们涵盖了所需的设置，请选择特定的插件）。

**video**

在视频卡上设置各种powersave级别（目前仅支持Radeon卡）。可以使用*radeon_powersave*参数指定powersave级别。支持的值是：`default`，`auto`，`low`，`mid`，`high`，和`dynpm`。

bootloader

将参数添加到内核引导命令行。此插件支持旧版GRUB 1，GRUB 2以及带可扩展固件接口（EFI）的GRUB。可以通过*grub2_cfg_file*选项指定grub2配置文件的自定义非标准位置。参数将添加到当前grub配置及其模板中。需要重新启动计算机才能使内核参数生效。

可以通过以下语法指定参数：

``` 
cmdline= arg 1 arg 2 ... arg n。
```