# Docker存储

默认情况下，在荣情况内创建的所有文件都存储在可写容器层。这意味着：

- 当该容器不再存在时，数据不会持久保存，并且如果另一个进程需要它，则可能很难从容器中取出数据。
- 容器的可写层与运行容器的主机紧密耦合。不能轻易将数据移动到其他地方。
- 写入容器的可写层需要存储驱动来管理文件系统。存储驱动程序使用LInux内核提供的联合文件系统。与使用直接写入主机文件系统的数据卷相比，额外的抽象降低了性能。

Docker为容器提供了两种选项来讲文件存储在主机中，以便即使在容器停止后文件也可以持久存储：`volumes`和`bind mounts`，如果是Linux，还可以使用`tmpfs`。Windows的话，则可以使用`named pipe`。

容器中的数据，它在容器的文件系统中显示为目录或单个文件。

`volumes`，`bind mounts`和`tmpfs mounts`之间的差异是数据在Docker主机上的位置。

![](https://blog-image.nos-eastchina1.126.net/types-of-mounts.png)

- **Volume**存储在由Docker管理的主机文件系统的一部分中（`/var/lib/docker/volumes/`）。非Docker进程不应修改文件系统的这一部分。Volume是在Docker中持久保存数据的最佳方法。
- **Bind mounts**可以存储在主机系统上的任何位置。甚至可以是重要的文件系统或目录。Docker主机或Docker容器上的非Docker进程可以随时对其进行修改。
- **tmpfs**挂载仅存储在主机系统的内存中，并且永远不会写入主机系统的文件系统中。

## Volume