# 使用 HostAliases 向 Pod /etc/hosts 文件添加条目

当DNS配置以及其它选项不合理的时候，通过向Pod的`/etc/hosts`文件中添加条目，可以在Pod级别覆盖对主机名的解析。用户可以通过PodSpec的HostAliases字段来添加这些自定义条目。

建议通过使用`HostAliases`来进行修改，因为该文件由`kubelet`管理，并且可以在Pod创建/重启过程中被重写。



## 默认hosts文件内容

让我们从一个Nginx Pod开始，给该Pod分配一个IP：

```bash
kubectl run nignx --image nginx --generator=run-pod/v1
```

检查Pod IP:

```bash
kubectl get pods --output=wide
```

hosts文件内容如下：

```bash
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

默认，hosts 文件只包含 ipv4 和 ipv6 的样板内容，像 `localhost` 和主机名称。

## 通过 HostAliases 增加额外的条目

除了默认的样板内容，我们可以向hosts文件添加额外的条目，将`foo.local`、`bar.local`解析为`127.0.0.1`，将`foo.remote`、`bar.remote`、解析为`10.1.2.3`，我们可以在`.spec.hostAliases`下为Pod添加`HostAliases`。

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: hostaliases-pod
spec:
  restartPolicy: Never
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
  containers:
  - name: cat-hosts
    image: busybox
    command:
    - cat
    args:
    - "/etc/hosts"
```

可以使用以下命令启动此Pod：

```bash
kubectl apply -f hostaliases-pod.yaml
```

检查状态：

```bash
kubectl get po -o wide
```

 hosts 文件的内容看起来类似如下这样： 

```bash
kubectl logs hostaliases-pod
```

```bash
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.5	hostaliases-pod

# Entries added by HostAliases.
127.0.0.1	foo.local	bar.local
10.1.2.3	foo.remote	bar.remote
```

 在最下面额外添加了一些条目。 

## 为什么kubelet管理hosts文件？

`kubelet`管理Pod中每个容器的hosts文件，避免Docker在容器已经启动之后去修改该文件。

因为该文件是托管性质的文件，无论容器重启或Pod重新调度，用户修改该hosts文件的任何内容，都会在kubelet重新安装后背覆盖。因此不建议修改该文件内容。

