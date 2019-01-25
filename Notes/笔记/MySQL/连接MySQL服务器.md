# 连接MySQL服务器

如何建立与`MySQL`服务器的连接，要使用客户端程序连接到`MySQL`服务器，需使用正确的连接参数，例如，运行服务器的主机的名称以及用户名和密码。每个连接参数都有一个默认值，可以根据需要使用命令行或配置文件中指定值来覆盖默认选项。

**不指定任何参数**

```bash
mysql
```

由于没有参数，因此使用默认值：

- 默认主机名是`localhost`，这在Unix上有特殊含义。
- 在Win上的默认用户名为`ODBC`上，在Unix上为当前登录名。
- 没有默认密码
- 对于`MySQL`第一个`nonoption`参数被视为默认数据库名称。如果没有这样的选项，`MySQL`不会选择默认数据库。

要明确指定主机名和用户名以及密码，指定以下选项：

```bash
mysql --host=localhost --user=myname --password=password mydb
mysql -h localhost -u myname -ppassword mydb
```

密码选项使可选的：

- 如果使用`-p`或`–password`选项，并指定密码的值，`-p`必须没有空格的情况下紧跟密码。
- 如果使用`-p`或`–password`选项，但未明确指定密码值，则客户端程序会提示输入密码。输入密码时不会显示。这比明确指定密码值更安全。

```bash
mysql --host=localhost --user=myname --password mydb
mysql -h localhost -u myname -p mydb
```

在Unix上，`MySQL`程序会特别处理`localhost`，其方式可能与期望的与其他基于网络的程序不同。

客户端确定要简历连接类型如下：

- 如果未指定主机或是`localhost`，则假定与本地主机连接：
  - 在WIN上，如果服务器启用了共享内存连接，则客户端使用共享内存进行连接。
  - 在Unix上，客户端使用Unix套接字文件进行连接。`–socket`选项或`MYSQL_UNIX_PORT`环境变量可用于指定套接字名称。
- 在WIN上，如果`host`是`.`，或`TCP/IP`未启用`–socket`且未指定或主机为空，则客户端使用命名管道进行连接。
- 否则使用`TCP/IP`

`--protocol`选项可以建立特定类型的连接，即使其他选项通常默认认为某些其他协议。也就是说，`--protocol`可以明确指定连接协议并覆盖前面的规则，即使对于`localhost`。



