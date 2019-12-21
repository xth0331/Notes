# Ansible的简介方法和细节

## ControlPersist和paramiko

默认情况下，Ansible使用本机`OpenSSH`，因为它支持**ControIPersist** ，`Kerberos`和`~/.ssh/config`中的选项（如Jump Host设置）。如果控制节点使用不支持**ControlPersist**的较旧版本的`OpenSSH`，则Ansible将回退到称为`paramiko`的`OpenSSH`的Python实现。

## SSH密钥设置

默认情况下，Ansible嘉定正在使用SSH密钥连接到远程计算机。鼓励使用SSH密钥，但是如果需要，可以使用`--ask-pass`选项进行密码验证。如果需要提供密码进行特权升级（sudo，pbrun等），请使用`--ask-become-pass`选项。

