# 配置OPENSSH

## 配置文件

有两套不同的配置文件：客户端程序（ssh，scp和sftp），以及SSH服务器（
sshd守护进程）。

系统范围的SSH配置信息存储在`/etc/ssh/`目录中，用户特定的ssh配置信息存储在`~/.ssh/`目录中。

> 系统范围的配置文件

| 文件                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `/etc/ssh/moduli`                  | 包含用于Diffie-Hellman密钥交换的Diffie-Hellman组，这对用于构建安全传输层至关重要。在SSH会话开始时交换密钥时，会创建一个共享的秘密值，这个值无法由任何一方单独确定，此值用于提供主机身份验证。 |
| `/etc/ssh/ssh_config`              | 默认的SSH客户端配置文件。请注意，如果`~/.ssh/config`存在，它将被覆盖。 |
| `/etc/ssh/sshd_config`             | ssd守护进程的配置文件。                                      |
| `/etc/ssh/ssh_host_ecdsa_key`      | sshd守护进程使用的ECDSA私钥。                                |
| `/etc/ssh/ssh_ho;st_ecdsa_key.pub` | sshd守护进程使用的ECDSA公钥                                  |
| `/etc/ssh/ssh_host_rsa_key`        | sshd守护进程用于使用SSH V2的RSA私钥。                        |
| `/etc/ssh/ssh_host_rsa_key.pub`    | sshd守护进程用于使用SSH V2的RSA公钥。                        |
| `/etc/pam.d/sshd`                  | sshd守护进程的PAM配置文件。                                  |
| `/etc/sysconfig/sshd`              | sshd服务的配置文件。                                         |

> 用户特定的配置文件

| 文件                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `~/.ssh/authorized_keys` | 保存服务器的授权公钥列表。当客户端连接到服务器时，服务器通过检查存储在此文件的以签名公钥来验证客户端。 |
| `~/.ssh/id_ecdsa`        | 包含用户的ECDSA私钥。                                        |
| `~/.ssh/id_ecdsa.pub`    | 用户的ECDSA公钥。                                            |
| `~/.ssh/id.rsa`          | ssh 使用的SSH V2 RSA私钥。                                   |
| `~/.ssh/id.rsa.pub`      | ssh 使用的SSH V2 RSA公钥。                                   |
| `~/.ssh/known_hosts`     | 包含用户访问的SSH服务器的主机密钥。此文件对于确保ssh客户端连接到正确的SSH服务器非常重要。 |



## 启动OpenSSH服务器

要运行OpenSSH服务器，必须安装`openssh-server`软件包。

要在当前会话中启动sshd守护进程，请在shell提示符输入以下内容：

```bash
systemctl start sshd.service
```

要在当前会话中停止正在运行的sshd守护进程，请使用：

```bash
systemctl stop sshd.service
```

如果希望守护进程在引导是启动：

```bash
systemctl enable sshd.service
```

sshd守护进程取决于`network.target`目标单位，这是足够的静态配置的网络接口，并为默认`ListenAddress 0.0.0.0` 选项。要在`ListenAddress` 指令中指定不同的地址并使用较慢的动态网络配置，请将`network-online.target`目标单元的依赖性添加到`sshd.service`单元文件中。实现此目的，请在`/etc/systemd/system/sshd.service.d/local.conf`键入以下内容：

```bash
[Unit]
Wants=network-online.target
After=network-online.target
```

之后，使用以下命令重新加载`systemd`管理器配置：

```bash
systemctl deamon-reload
```

请注意，如果重新安装系统，将创建一组新的标识。因此，在重新安装之前使用任何OpenSSH工具连接到系统的客户端将看到以下消息：

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
```

为了防止这种情况，可以从；那个`/etc/ssh`目录中备份相关文件。并在重新安装系统时还原文件。

## 需要SSH进行远程连接

要是SSH真正有效，应禁止使用不安全的连接协议，否则，用户的密码可能会在一个会话中使用SSH进行保护，只能在以后使用Telnet登录时截获。需要禁用的服务包括`telnet`,`rsh`,`rlogin`和`vsftpd`。



## 使用基于密钥的身份验证

要进一步提高系统安全性，请生成SSH密钥对，然后通过禁用密码身份验证来强制执行基于密钥的身份验证。为此，请编辑`/etc/ssh/sshd_config`然后更改`PaswordAuthentication`选项值：

```bash
PasswordAuthentication no
```

如果你不是一个新的默认安装其他的系统上工作，检查`PubkeyAuthentication no` 是否设置，如果远程连接，不使用控制台访问，建议在禁用密码验证之间测试基于密钥的登录。

为了能够使用`ssh`，`scp`或`sftp`从客户端计算机连接到服务器，请按照以下步骤生成授权密钥对，请注意，必须分别为每个用户生成密钥。

要对安装NFS的主目录使用基于密钥的身份验证，首先设置SELinux：

```bash
setsebool -P use_nfs_home_dirs 1
```

##　生成密钥对

要为SSH V2生成RSA密钥对，请按照下列步骤操作：

1. 通过在shell提示符下键入一下内容来生成RSA密钥对：

  ```bash
  ssh-keygen -t rsa
  ```

2. 输入`Enter`确认创建密钥的默认位置（~/.ssh/id_rsa）。

3. 输入密码（可选），在此之后，将看到类似以下消息。、

   ```bash
   Your identification has been saved in /home/USER/.ssh/id_rsa.
   Your public key has been saved in /home/USER/.ssh/id_rsa.pub.
   The key fingerprint is:
   SHA256:UNIgIT4wfhdQH/K7yqmjsbZnnyGDKiDviv492U5z78Y USER@penguin.example.com
   The key's randomart image is:
   +---[RSA 2048]----+
   |o ..==o+.        |
   |.+ . .=oo        |
   | .o. ..o         |
   |  ...  ..        |
   |       .S        |
   |o .     .        |
   |o+ o .o+ ..      |
   |+.++=o*.o .E     |
   |BBBo+Bo.  oo     |
   +----[SHA256]-----+
   ```

4. 默认情况下，`~/.ssh/`目录的权限设置为`rwx------`或700八进制表示法表示。这是为了确保只有*USER*才能查看内容。如果需要，可以使用一下命令确认：

   ```bash
   ls -ld ~/.ssh
   drwx------. 2 USER USER 54 Nov 25 16:56 /home/USER/.ssh/
   ```

5. 要将公钥复制到远程计算机，请按一下格式：

   ```bash
   ssh-copy-id user@hostname
   ```

`~/.ssh/id*.pub`如果尚未安装，则会复制最近修改过的公钥，或者指定公钥的文件名，如下所示：

```bash
ssh-copy-id -i ~/.ssh/id.rsa.pub user@hostname
```

这将复制 `~/.ssh/id_rsa.pub`到`~/.ssh/authorized_keys`文件到您要连接的机器上。如果文件已存在，则将键附加到其末尾。

要为SSH V2生成ECDSA密钥对，请按照下列步骤操作：

1. 通过在shell提示符下键入以下内容来生成ECDSA密钥对：

   ```bash
   ssh-keygen -t ecdsa
   ```



### 配置ssh-agent

要存储密码，以便每次启动与远程计算机的连接时都不需要密码，这可以使用`ssh-agent`身份验证代理。

确保安装`openssh-askpass`

要保存某个shell提示符的密码，请使用以下命令：

```bash
ssh-add
```

注意，注销时，密码将被遗忘，每次登陆虚拟控制台或终端窗口时都必须执行该命令。

