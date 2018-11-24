# OpenSSH客户端

要从客户端连接到OpenSSH服务器，必须安装`openssh-clients`软件包。

## 使用ssh

ssh允许您登陆到远程计算机并在那里执行命令，替代`rlogin` ,`telnet`程序。

与`telnet`命令类似，使用一下命令登陆到远程主机：

```bash
ssh hostname
```

例如，要登陆主机名为`ssh.example.com`，请在shell提示符下键入以下内容：

```bash
ssh ssh.example.com
```

这将使用您在本地计算机上使用的相同用户名登陆。如果要指定其他用户名，请使用一下格式的命令：

```bash
ssh user@hostname
```

例如：

```bash
ssh root@ssh.example.com
```

第一次启动连接时，将显示类似于以下内容的消息：

```bash
The authenticity of host 'ssh.example.com' can't be established.
ECDSA key fingerprint is SHA256:vuGKK9dsW34zrZzwjl5g+vOE6EZQvHRQ8zObKYO2mW4.
ECDSA key fingerprint is MD5:7e:15:c3:03:4d:e1:dd:ee:99:dc:3e:f4:b9:67:6b:62.
Are you sure you want to continue connecting (yes/no)?
```

在回答此对话框中的问题之前，用户应始终检查指纹是否正确。用户可以要求服务器的管理员确认密钥是否正确。这应该以安全且事先商定的方式进行，如果用户可以访问服务器的主机密钥，则可以使用`ssh-kengen`命令检查：

```bash
ssh-kengen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
```

> 要获取MD5密钥指纹，请使用 -E md5选项

```bash
ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub -EM md5
```

> 如果SSH服务器的主机密钥更改，则客户端会通知用户连接无法继续，直到从`~/.ssh/known_hosts`文件中删除服务器的主机密钥。但是，在执行此操作之前，请与SSH服务器的系统管理员联系以验证服务器是佛收到损害。
>
> 要从`~/.ssh/known_hosts`文件中删除密钥，请发出如下命令：
>
> ```bash
> ssh-keygen -R ssh.example.com
> ```

ssh验证完成后， 将为您提供远程计算机的shell提示符。

或者，该ssh程序可用于在远程计算机上执行命令，而无需到shell提示符：

```bash
ssh root@hostname command
```

例如：

```bash
ssh root@ssh.example cat /etc/redhat-release
```

## 使用scp

`scp` 可用于通过安全的加密连接在计算机之间传输文件。在它的设计中，非常类似`rcp`要将本地文件传输到远程系统，使用一下格式的命令：

```bash
scp localfile username@hostname:remotefile
```

例如，如果要复制`taglist.vim`到远程计算机ssh.example.com:

```bash
scp taglist.vim  USER@ssh.example.com:.vim/plugin/taglist.vim
```

要将远程文件传输到本地系统：

```bash
scp username@hostname:remotefile localfile
```

例如将远程主机的`.vimrc`复制到本地：

```bash
scp USER@ssh.example.com:.vimrc .vimrc
```

## 使用sftp

`sftp`可用于打开安全的交互式FTP会话。在其设计中，它类似于`ftp`使用安全的加密连接。

要连接到远程系统，请使用以下格式的命令：

```bash
sftp username@hostname
```

例如，登录到名为远程机器`ssh.example.com`与`USER`作为用户名，类型：

```bash
sftp USER@ssh.example.com
```

**可用的sftp命令**

| 命令                               | 描述                                                       |
| ---------------------------------- | ---------------------------------------------------------- |
| `ls`[ *目录* ]                     | 列出远程*目录*的内容。如果未提供，则默认使用当前工作目录。 |
| `cd` *目录*                        | 将远程工作目录更改为*目录*。                               |
| `mkdir` *目录*                     | 创建一个远程*目录*。                                       |
| `rmdir` *路径*                     | 删除远程*目录*。                                           |
| `put` *localfile* [ *remotefile* ] | 将*本地文件*传输到远程计算机。                             |
| `get` *remotefile* [ *localfile* ] | 传输*参数】remotefile*从远程机器。                         |