# Docker架构简单概述



## Docker Engine

Docker Engine 是具有以下组件的客户端-服务器应用程序：

- 服务器是长期运行的程序，守护进程(`dockerd`)。
- REST API，它指定程序可以用来与守护进程进行通信并指示其操作的接口。
- 命令行界面（CLI）客户端（`docker`）。

![](https://blog-image.nos-eastchina1.126.net/engine-components-flow.png)

CLI使用Docker REST API 通过脚本或直接CLI命令控制或与Docker守护进程交互。

## 架构

Docker使用客户端-服务器架构。Docker *客户端*与Docker *守护进程*进行对话，该*守护进程*完成了构建，运行和分发Docker容器的繁重工作。Docker客户端和守护程序*可以* 在同一系统上运行，或者您可以将Docker客户端连接到远程Docker守护程序。Docker客户端和守护程序在UNIX套接字或网络接口上使用REST API进行通信。

![](https://blog-image.nos-eastchina1.126.net/architecture.svg)

### Docker守护进程

`dockerd`监听Docker API请求并管理Docker对象，例如，image，container，network和volume。守护进程还可以与其他守护进程通讯以管理Docker服务。

### Docker客户端

`docker cli`

### Docker registries

存储image

## 底层技术

Go编写，利用LInux内核功能交付。

### Namespaces

Docker使用`namespaces`提供容器隔离工作区的技术。运行容器时，Docker会为该容器创建一组namespaces。

Docker Engine在Linux上使用以下namespaces：

- **`pid`命名空间：**进程隔离（PID：进程ID）。
- **`net`命名空间：**管理网络接口（NET：网络）。
- **`ipc`命名空间：**管理访问IPC资源（IPC：进程间通信）。
- **`mnt`命名空间：**管理文件系统挂载点（MNT：Mount）。
- **`uts`命名空间：**隔离内核和版本标识符。（UTS：Unix时间共享系统）。

### Control groups

Linux的Docker Engine有害以来另一种名为`cgroups`的技术。cgroup将应用程序限制为一组特定的资源。允许Docker Engine将可用的硬件资源共享给容器，并有选择地限制和约束。

### Union file systems

UnionFS是通过创建层来进行操作的文件系统，非常轻量，快速。使用UnionFS为容器提供构建模块。可以使用多种UnionFS，包括AUFS，btrfs，vfs和DeviceMapper。

### 容器格式

DockerEngine将namespaces，cgroups和UniosFS组合到一个称为容器的包装器。默认格式为`libcontainer`。

