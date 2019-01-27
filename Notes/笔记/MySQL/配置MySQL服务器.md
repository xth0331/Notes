# 配置MySQL服务器

MySQL服务器`mysqld`有许多命令选项和系统变量，可以在启动时设置它们以配置其操作。要确定服务器使用的缺省选项和系统变量值，执行：

```bash
mysqld --verbose --help
```

要查看服务器在运行时实际使用的当前系统变量值，可以连接到服务器后执行以下语句：

```mysql
SHOW VARIABLES;
```

要查看正在运行的服务器的某些统计和状态指示，可以连接到服务器后执行以下语句：

```mysql
SHOW　STATUS;
```

也可以使用`mysqladmin`命令：

```bash
mysqladmin variables 
mysqladmin extended-status
```

> 可以从`Performance Schema`获得更详细的监控信息，另外，MySQL `sys Schema`是一组对象可以访问性能模式收集的数据。



##  