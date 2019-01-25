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

仅使用或检查与所选协议相关的连接选项。其他连接选项将被忽略。例如，`--host=localhost`在Unix上，客户端尝试使用Unix套接字文件连接本地服务器。即使给出了指定端口的选项。

要确保客户端与本地服务器建立`TCP/IP`连接，请使用`--host`或`-h`指定主机名值`127.0.0.1`，或本地服务器的IP地址或名称。也可以指定`localhost`使用`--protocol=TCP`明确指定连接协议：

```bash
mysql --host=127.0.0.1
mysql --protocol=TCP
```

与远程服务器的连接始终使用TCP / IP。使用默认端口号连接到运行的服务器`remote.example.com` （3306）：

```bash
mysql --host=remote.example.com
```

指定端口，使用`--port`或`-P`选项：

```bash
mysql --host=remote.example.com --port=3307
```



以下列表总结可用于客户端如何连接到服务器的选项：

- `–default-auth=PLUGIN`

  身份验证插件

- `--host=HOST_NAME`,`-h HOST_NAME`

  指定连接的MySQL服务器，默认`localhost`

- `--password[=PAAS_VAL]`,`-p[PASS_VAL]`

  MySQL账户的密码。密码值可选，但是如果指定，密码值与`-p`之间没有空格。

- `--port=PORT_NUM`,`-P PORT_NUM`

  用于连接的端口号，用于使用TCP/IP建立的连接。默认为`3306`

- `--protocol={TCP|SOCKET|PIPE|MEMORY}`

  指定用于连接服务器的协议

  下表显示了允许的 [`--protocol`](https://dev.mysql.com/doc/refman/5.7/en/connecting.html#option_general_protocol)选项值，并指出了可以使用每个值的平台。值不区分大小写。

  | `--protocol`值 | 连接协议                         | 允许的操作系统 |
  | -------------- | -------------------------------- | -------------- |
  | `TCP`          | 到本地或远程服务器的TCP / IP连接 | 所有           |
  | `SOCKET`       | Unix套接字文件连接到本地服务器   | 仅限Unix       |
  | `PIPE`         | 与本地或远程服务器的命名管道连接 | 仅限Windows    |
  | `MEMORY`       | 与本地服务器的共享内存连接       | 仅限Windows    |

- `--socket=FILE_NAME`,`-S FILE_NAME`

  在Unix上，要是用的Unix套接字文件名称，用于连接本地服务器。默认值为`/tmp/mysql.sock`

- `--ssl`

  如果配置了支持SSL，则使用SSL建立与服务器的安全连接

- `--tls-version=PROTOCOL_LIST`

  客户端允许加密连接的协议。值是一个或多个以逗号分隔的协议名称的列表。

- `--user=USER_NAME`,`-u USER_NAME`

  要使用的MySQL的用户名。win上默认为ODBC的，Unix上默认为登录的用户。



可以指定在建立连接时使用的默认值，这样每次调用客户端程序都无需在命令行输入。有以下几种方式：

- 可以在配置文件中`[clietn]`部分指定连接参数。

  ```bash
  [client]
  host=host_name
  user=user_name
  password=your_pass
  ```

- 可以使用环境变量指定一些连接参数。