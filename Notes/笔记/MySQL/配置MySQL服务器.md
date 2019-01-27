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

  | 属性                | 值                                |
  | ----------------------- | ------------------------------------ |
  | **命令行格式** | `--authentication-windows-log-level` |
  | **系统变量**   | `authentication_windows_log_level`   |
  | **范围**            | Global                               |
  | **动态修改**         | 不支持                             |
  | **类型**                | 整数                          |
  | **默认值**    | `2`                                  |
  | **最小值**       | `0`                                  |
  | **最大值**       | `4`                                  |

  当`authentication_windows`启用Windows身份插件并启用调试代码时，此变量才可用。

  此变量设置Windows身份验证插件的日志记录级别。下表显示了允许的值。

  | 值   | 描述                  |
  | ---- | --------------------- |
  | 0    | 没有记录              |
  | 1    | 仅记录错误消息        |
  | 2    | 记录1级消息和警告消息 |
  | 3    | 记录2级消息和信息说明 |
  | 4    | 记录3级消息和调试消息 |

- `authentication_windows_use_principal_name`

  | 属性           | 值                                            |
  | -------------- | --------------------------------------------- |
  | **命令行格式** | `--authentication-windows-use-principal-name` |
  | **系统变量**   | `authentication_windows_use_principal_name`   |
  | **范围**       | Global                                        |
  | **动态**       | 不支持                                        |
  | **类型**       | 布尔                                          |
  | **默认值**     | `ON`                                          |

  使用InitSecurityContext()函数进行身份验证的客户端应提供表示其连接服务的字符串（**targetName**）。MySQL使用运行服务器的账户的主体名称（UPN）。UPN的格式为`user_id@computer_name`无需在任何地方注册即可使用。

  此变量控制服务器是否在初始质询中发送UPN。默认情况下，该变量已启用。出于安全原因，可以禁用它以避免以明文形式将服务器的帐户名发送给客户端。如果禁用该变量，则服务器始终`0x00`在第一个质询中发送一个 字节，客户端不指定*targetName*，因此使用NTLM身份验证。

  如果服务器无法获取其UPN（主要发生在不支持Kerberos身份验证的环境中），则服务器不会发送UPN并使用NTLM身份验证。

