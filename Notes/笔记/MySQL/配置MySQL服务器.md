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



## MySQL服务器系统变量

MySQL服务器维护许多配置其操作的系统变量。每个系统变量都有一个默认值。可以使用命令行选项或配置文件中的选项在服务启动时设置系统变量。它们中的大多数都是可以在运行时使用`SET`语句动态更改的，这使修改服务器的操作不需重启。还可以在表达式中使用系统变量值。

在运行时，设置全局系统变量值需要`SUPER`权限。设置会话系统变量通常不需要特殊权限，可以由任何用户完成，但也有例外。

- `authentication_windows_log_level`

  | Property                | Value                                |
  | ----------------------- | ------------------------------------ |
  | **Command-Line Format** | `--authentication-windows-log-level` |
  | **System Variable**     | `authentication_windows_log_level`   |
  | **Scope**               | Global                               |
  | **Dynamic**             | No                                   |
  | **Type**                | Integer                              |
  | **Default Value**       | `2`                                  |
  | **Minimum Value**       | `0`                                  |
  | **Maximum Value**       | `4`                                  |