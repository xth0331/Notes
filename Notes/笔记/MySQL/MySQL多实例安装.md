# MySQL多实例安装

某些情况下，可能希望在一台机器上安装多个MySQL实例，可能希望测试新的MySQL版本，同时保持现有生产设置不受干扰。或希望让不同的用户访问他们自己管理的不同`mysqld`服务器。

每个实例可以使用不同的MySQL服务器二进制文件，或者对多个实例使用相同的二进制位文件，但无论是否使用不同的二进制文件，运行的每个实例必须配置一些参数为唯一值。消除实例间冲突。

> MySQL实例管理的主要资源时数据目录，每个实例都应使用不同的数据目录。

除了不同的数据目录之外，以下几个参数必须为每个服务器实例配置不同的值：

- `--port=PORT_NUMBER`

  TCP/IP连接的端口号。或者，如果你的主机具有多个网络地址，则可以使用`--bind-address`指定每个服务器实例监听不同的IP地址。

- `–-socket=FILE_NAME`

  `--socket`控制Unix上的Unix套接字文件路径。必须指定为不同文件。

- `--pid-file=FILE_NAME`

  指定服务器在其中写入其进程ID的文件的路径名。

如果使用以下日志文件选项，则每个实例的值必须不同：

- `–general_log_file=FILE_NAME`
- `--log-bin=FILE_NAME`
- `--slow_query_log_file=FILE_NAME`
- `--log-error=FILE_NAME`



事先准备好已经安装单实例的mysql

现在演示通过mysqld_multi 在一台机器上管理四个实例，实例信息如下：

| 实例名称  | 端口   | 数据目录 | socket             |
| --------- | ------ | -------- | ------------------ |
| `mysqld1` | `3306` | `/data1` | `/tmp/mysql.sock1` |
| `mysqld2` | `3307` | `/data2` | `/tmp/mysql.sock2` |
| `mysqld3` | `3308` | `/data3` | `/tmp/mysql.sock3` |
| `mysqld4` | `3309` | `/data4` | `/tmp/mysql.sock4` |

首先停止数据库。然后修改配置文件my.cnf

以下是`my.cnf`部分内容

```bash
[client]

[mysqld_multi]
mysqld = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
log = /usr/local/mysql/multi.log
user = root
pass = password         # 5.7需要，否则无法停止数据库实例，5.6需要配置为password = password

[mysqld1]
datadir = /data1
socket = /tmp/mysql.sock1
port = 3306
user = mysql
performance_schema = off
innodb_buffer_pool_size = 32M
bind_address = 0.0.0.0
skip_name_resolve
[mysqld2]
datadir = /data2
socket = /tmp/mysql.sock2
port = 3307
user = mysql
performance_schema = off
innodb_buffer_pool_size = 32M
bind_address = 0.0.0.0
skip_name_resolve
[mysqld3]
datadir = /data3
socket = /tmp/mysql.sock3
port = 3308
user = mysql
performance_schema = off
innodb_buffer_pool_size = 32M
bind_address = 0.0.0.0
skip_name_resolve
[mysqld4]
datadir = /data4
socket = /tmp/mysql.sock4
port = 3309
user = mysql
performance_schema = off
innodb_buffer_pool_size = 32M
bind_address = 0.0.0.0
skip_name_resolve

```

创建所需的数据目录

```bash
mkdir -pv /data{1..4}
chown -R mysql:mysql /data{1..4}
```

接下来，如果已经有数据库二进制文件，可以直接复制mysql库文件到不同实例的数据目录下；如果没有就需要重新初始化各个数据库实例（演示需要，我就直接复制库目录）:

```bash
echo /data{1..4} | xargs -n 1 cp -rp /data/mysql_data/mysql/
```

测试：

```bash
mysqld_multi start 
mysqld_multi report
```

复制启动脚本：

```bash
cp /usr/local/mysql/support-files/mysqld_multi.server /etc/init.d/mysqld_multi
/etc/init.d/mysqld_multi stop
/etc/init.d/mysqld_multi report 
/etc/init.d/mysqld_multi start 
/etc/init.d/mysqld_multi report
```

如果部署多实例出现问题，请检查error_log 

基于socket的连接或者基于tcp的连接都可以，前提是有这个权限；进入不同的实例。查看对应的变量来确定是否为不同实例。

```bash
 mysql -u root -p -S /tmp/mysql.sock2 
```

查看对应实例的变量：

```mysql
show variables like "socket";
show variables like "port"; 
show variables like "datadir";
```

