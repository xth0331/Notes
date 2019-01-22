# 通用二进制MySQL升级

> 示例中是从MySQL5.6升级到MySQL5.7。
>
> 注： MySQL的安装文件和数据文件应分离存储。

## 环境信息

- MySQL安装目录：
  - **MySQL5.6:** /usr/local/mysql-5.6.42-linux-glibc2.12-x86_64
  - **MySQL5.7:**/usr/local/mysql-5.7.24-linux-glibc2.12-x86_64
- datadir目录：
  - /data/mysql_data/



## 升级数据库

```bash
/etc/init.d/mysqld stop
cd /usr/local/
unlink mysql
ln -sv mysql-5.7.24-linux-glibc2.12-x86_64 mysql
cd /usr/local/msyql
chown -R root.mysql .
cp -r /data/mysql/data/mysql /backup/mydql-5.6.42,backup
/etc/init.d/mysqld start 
mysql_upgrade -s 
```

