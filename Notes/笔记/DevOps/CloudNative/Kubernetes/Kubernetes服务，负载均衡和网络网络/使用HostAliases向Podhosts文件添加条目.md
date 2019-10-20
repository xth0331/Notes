# 使用 HostAliases 向 Pod /etc/hosts 文件添加条目

当DNS配置以及其它选项不合理的时候，通过向Pod的`/etc/hosts`文件中添加条目，可以在Pod级别覆盖对主机名的解析。用户可以通过PodSpec的HostAliases字段来添加这些自定义条目。

建议通过使用`HostAliases`来进行修改，因为该文件由`kubelet`管理，并且可以在Pod创建/重启过程中被重写。



## 默认hosts文件内容

