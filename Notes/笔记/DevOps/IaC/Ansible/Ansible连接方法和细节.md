# Ansible的简介方法和细节

## ControlPersist和paramiko

默认情况下，Ansible使用本机`OpenSSH`，因为它支持**ControIPersist** ，`Kerberos`和`~/.ssh/config`中的选项（如Jump Host设置）。如果控制节点使用不支持**ControlPersist**的较旧版本的`OpenSSH`，则Ansible将回退到称为`paramiko`的`OpenSSH`的Python实现。

## SSH密钥设置

默认情况下，Ansible嘉定正在使用SSH密钥连接到远程计算机。鼓励使用SSH密钥，但是如果需要，可以使用`--ask-pass`选项进行密码验证。如果需要提供密码进行特权升级（sudo，pbrun等），请使用`--ask-become-pass`选项。



> 当使用ssh连接插件（默认设置）时，Ansible不会公开允许用户和ssh进程之间通信的通道，以手动接受密码来解密ssh密钥。强烈建议使用`ssh-agent`。

要设置SSH agent以避免重新输入密码，可以执行以下操作：

```bash
ssh-agent bash
ssh-add ~/.id_rsa
```

根据设置，可能希望使用Ansible的`--private-key`私钥命令选项来指定一个`pem`文件，可以添加私钥文件：

```bash
ssh-agent bash
ssh-add ~/.ssh/keypair.pem
```

## 针对localhost运行

可以通过使用`localhost`或`127.0.0.1`作为服务器名称来对控制节点运行命令：

```bash
ansible localhost -m ping -e 'ansible_python_interpreter="/usr/bin/env python"'
```

可以通过将`localhost`添加到清单文件中来显式指定`localhost`：

```ini
localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python"
```

## 主机密钥检查

Ansible默认启用主机密钥检查。检查主机密钥可以防止服务器欺骗和中间人攻击，但确实需要进行一些维护。

如果主机被重新安装并且在`known_hosts`中具有不同的密钥，这将导致错误消息，直到更正为止。如果新主机不在`known_hosts`中，则控制节点可能回提示确认密钥，如果使用来自cron的Ansible，这将需要交互完成。

如果了解其中含义并希望禁用此行为，则可以通过编辑`/etc/ansible/ansible.cfg`或`~/.ansible.cfg`执行以下操作：

```ini
[defaults]
host_key_checking = False
```

或者，可以通过`ANSIBLE_HOST_KEY_CHECKING=False`环境变量：

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

另请注意，在`paramiko`模式下检查主机密钥的速度相当慢，因此，在使用此功能的时候，建议使用`ssh`。

## 其他链接方法

Ansible可以使用SSH以外的各种连接方法。您可以选择任何连接插件，包括在本地管理事物以及管理chroot，lxc和jail容器。一种称为“ ansible-pull”的模式还可以反转系统，并通过计划的git checkout将系统置为“phone home”，以从中央存储库中提取配置指令。

