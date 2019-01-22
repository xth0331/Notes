# MySQL通用二进制安装

> 不介绍win的安装方法，本次安装环境为CentOS7

## 下载

在[MySQL官网](https://dev.mysql.com/downloads/mysql/5.6.html#downloads)下载对应的安装包，

将`mysql-xxxx.tar.gz`安装包放在需要安装MySQL的文件系统目录下，例如/opt,

> 在安装之前先确认系统中是否存在`/etc/my.cnf`等mysql可读的配置文件。如果存在请提前移除。

> MySQL依赖于`libaio`库，如果未安装，请安装。
>
> ```bash
> yum install libaio
> ```

## MySQL安装目录布局

> MySQL通常放在`/usr/local/mysql`。

**通用二进制包的MySQL（5.6）安装目录布局**

| 目录            | 目录的内容                             |
| --------------- | -------------------------------------- |
| `bin`,`scropts` | `mysqld`服务器，客户端和实用二进制程序 |
| `data`          | 日志文件，数据库                       |
| `docs`          | 日志文件，数据库                       |
| `include`       | 信息格式的MySQL手册                    |
| `lib`           | lib库                                  |
| `mysql-test`    | 测试套件                               |
| `man`           | Unix手册                               |
| `share`         | 用于数据库安装的错误小，字典和SQL      |
| `sql-bench`     | 基准测试                               |
| `support-files` | 其他支持文件，包括示例配置文件         |

**通用二进制包的MySQL（5.7）安装目录布局**

| 目录            | 目录的内容                          |
| --------------- | ----------------------------------- |
| `bin`           | `mysqld`服务器，客户端和实用程序    |
| `docs`          | 信息格式的MySQL手册                 |
| `man`           | Unix手册页                          |
| `include`       | 包含（标题）文件                    |
| `lib`           | lib库                               |
| `share`         | 用于数据库安装的错误消息，字典和SQL |
| `support-files` | 其他支持文件，包括示例配置文件      |

## 安装MySQL数据库

### 创建一个mysql用户和组

如果系统没有mysql的用户和组，需要创建，如果不想使用mysql为对应的用户和组，请修改mysql为对应的自定义用户和组。

```bash
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```

### 解压通用二进制安装包(5.6)

将文件解压缩到`/usr/local`

```bash
cd /usr/local/
tar -zxvf mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz 
ln -s mysql-5.6.42-linux-glibc2.12-x86_64 /usr/local/mysql 
cd /usr/local/mysql
chown -R mysql .
chgrp -R mysql .
scripts/mysql_install_db --user=mysql
chown -R root .
chown -R mysql data
bin/mysqld_safe --user=mysql &
cp support-files/mysql.server /etc/init.d/mysqld
export PATH=/usr/local/mysql/bin:$PATH
```

### 解压通用二进制安装包(5.7)

> 5.7 安装上稍有不同

```bash
cd /usr/local/
tar -zxvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz 
ln -s mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql 
cd mysql
mkdir mysql-files
chmod 770 mysql-files
chown -R mysql .
chgrp -R mysql .
bin/mysqld --initialize --user=mysql # 密码输出到标准输出，如果配置了errorlog ，则输出到errorlog
bin/mysql_ssl_rsa_setup
chown -R root .
chown -R mysql data mysql-files
```





## 启动MySQL

```bash
/etc/init.d/mysqld start 
/etc/init.d/mysqld status 
netstat -tnlp | grep 3306
```



