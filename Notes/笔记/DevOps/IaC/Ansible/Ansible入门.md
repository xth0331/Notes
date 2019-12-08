# Ansible 入门

学习Ansible的工作原理、基本的Ansible命令和剧本：

- 从清单中选择执行的机器
- 通常通过SSH连接到这些计算机（网络设备或其他受管节点）
- 将一个或多个模块复制到远程计算机上开始执行

Ansible可以做更多的事情，但是在探究Ansible的所有强大配置,部署和编排功能之前,应该了解最常见的用例。

## 从清单中选择机器

Ansible从清单中读取有关要管理那些计算机的信息。尽管可以将IP地址传递给临时命令，但是仍需要通过清单来利用Ansible的全部灵活性和可复用性。

### 创建基本清单

对于此基本清单，请编辑或创建`/etc/ansible/hosts`并向其中添加一些远程系统。对于此示例，请使用IP地址或FQDN（FQDN天下第一）：

```bash
192.168.1.105
vc01.dc.example.com
vc02.dc.example.com
```

清单文件可以存储的内容远不止IP和FQDN.可以创建别名使用,使用主机变量设置的那个主机的变量值，或使用组变量为多个主机设置变量值。

## 连接到远程节点

Ansible通过SSH协议与远程计算机通信.默认情况下,Ansible使用本机`OpenSSH`并使用当前的用户名连接到远程计算机,就像SSH一样。

### 检查SSH连接

确认可以使用相同的用户名使用SSH连接到清单中的所有节点。如有必要，将SSH public key添加到这些系统上的`authorized_keys`文件中。

可以通过多种方式覆盖默认的远程用户名，这包括在命令行传递`-u`参数,在清单文件中设置用户信息,或设置环境变量。

## 复制并执行模块

连接后，Ansible会将命令和剧本所需的模块传输到远程计算机上以执行。

### 运行第一个Ansible命令 

使用`ping`模块ping清单文件中的所有节点:

```bash
ansible all -m ping
```

默认情况下，Ansible使用SFTP传输文件。如果要管理的机器或设备不支持SFTP，则可以在配置中切换到SCP模式。这些文件被放置在一个临时目录中并从那里执行。

如果您需要特权提升（sudo和类似权限）来运行命令，请传递`become`标志：

```bash
# as bruce
ansible all -m ping -u bruce
# as bruce, sudoing to root (sudo is default method)
ansible all -m ping -u bruce --become
# as bruce, sudoing to batman
ansible all -m ping -u bruce --become --become-user batman
```

