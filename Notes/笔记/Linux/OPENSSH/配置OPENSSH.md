# 配置OPENSSH

## 配置文件

有两套不同的配置文件：客户端程序（ssh，scp和sftp），以及SSH服务器（
sshd守护进程）。

系统范围的SSH配置信息存储在`/etc/ssh/`目录中，用户特定的ssh配置信息存储在`~/.ssh/`目录中。

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

