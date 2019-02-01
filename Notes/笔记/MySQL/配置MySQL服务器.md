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

- `atuocommit`

  | 属性           | 值                 |
  | -------------- | ------------------ |
  | **命令行格式** | `--autocommit[=#]` |
  | **系统变量**   | `autocommit`       |
  | **范围**       | Global, Session    |
  | **动态**       | 支持               |
  | **类型**       | 布尔               |
  | **默认值**     | `ON`               |

  自动提交模式。如果设置为1，对表的所有更改都会立即生效。如果设置为0，必须使用`COMMIT`接受事务或使用`ROOLBACK`取消它。如果`autocommit`为0并且将其更改为1，则MySQL会对任何打开的事务执行自动`COMMIT`。开始事务的另一种方法是使用`START TRANSACTION`或`BEGIN`语句。

  默认情况下。客户端连接以将`autocommit`设置为1。要使客户端以0为默认值，使用`--autocommit=0`来设置全局自动提交。要使用配置文件中的选项设置变量值。需如下配置：

  ```mysql
  [mysqld]
  autocommit=0
  ```

- `automatic_sp_privileges`

  | 属性         | 值                        |
  | ------------ | ------------------------- |
  | **系统变量** | `automatic_sp_privileges` |
  | **范围**     | Global                    |
  | **动态**     | 支持                      |
  | **类型**     | 布尔                      |
  | **默认值**   | `TRUE`                    |

  当此变量设置为1（默认）时，如果用户无法执行更改或删除事务，则服务器会自动向存储事务创建者授予`EXECUTE`和`ALTER ROURINE`。（`ALTER ROUTINE`需要权限才能删除事务）当事务被删除时，服务器也会自动从创建者中删除这些权限。如果设置为0，则服务器不会自动添加或删除这些权限。

  事务的创建者是用于其执行`CREATE`语句的账户。这可能与`DEFINER`与事务定义中命名的账户不同。

- `auto_generate_certs`

  | 属性           | 值                                 |
  | -------------- | ---------------------------------- |
  | **命令行格式** | `--auto-generate-certs[={OFF|ON}]` |
  | **系统变量**   | `auto_generate_certs`              |
  | **范围**       | Global                             |
  | **动态**       | 不支持                             |
  | **类型**       | 布尔                               |
  | **默认值**     | `ON`                               |

  如果服务器时使用`OPENSSL`编译的，则此变量可用。它控制服务器是否自动生成数据目录中的SSL密钥和证书文件（如果尚不存在）。

  启动时，如果`auto_generate_certs`启用了系统变量，则服务器会自动在数据目录中生成服务器端和客户端SSL证书和密钥文件，没有指定除`--ssl`之外的SSL选项，并且数据目录中缺少服务器端SSL文件。这些文件使用SSL启用安全客户端连接。

- `avoid_temporal_upgrade`

  | 属性           | 值                                  |
  | -------------- | ----------------------------------- |
  | **命令行格式** | `--avoid-temporal-upgrade={OFF|ON}` |
  | **系统变量**   | `avoid_temporal_upgrade`            |
  | **范围**       | Global                              |
  | **动态**       | 支持                                |
  | **类型**       | 布尔                                |
  | **默认值**     | `OFF`                               |

  此变量控制`ALTER TABLE`是否隐式升级发现为`5.6.4`之前版本的时间列（TIME，DATATIME和TIMESTAMP列，不支持小数秒精度）。升级此类列需要进行表重建，这可防止使用可能以其他方式应用于要执行的操作的快速更改。

  默认情况下禁用此变量。启用它会导致`ALTER TABLE`不重建时间列，从而能够利用可能的快速更改。

  > 这个变量以启用，将在以后的版本中删除。

- `back_log`

  | 属性         | 值                                       |
  | ------------ | ---------------------------------------- |
  | **系统变量** | `back_log`                               |
  | **范围**     | Global                                   |
  | **动态**     | 不支持                                   |
  | **类型**     | 整数                                     |
  | **默认值**   | `-1` （表示自动调整大小;不指定此文字值） |
  | **最小值**   | `1`                                      |
  | **最大值**   | `65535`                                  |

  MySQL可以拥有的未完成连接请求的数量。当MySQL主线程在很短的时间内获得很多连接请求时，这就会发挥作用。然后，主线程检查连接并启动新线程需要一些时间（很少）。`back_log`值表示在MySQL暂时停止应答新请求之前的短时间内可以堆叠的请求数。只有在短时间内预期有大量连接时，才需要增加此值。

  换句话说，此值是传入`TCP/IP`连接的监听队列大小。操作系统对此队列大小有自己的限制。Unix `listen()`系统调用的手册有更多细节。检查操作系统文档以获取此变量的最大值。`back_log`的值不能高于操作系统限制。

  默认值基于以下公式，上限为900：

  ```bash
  50 + (max_connections/5)
  ```

- `basedir`

  | 属性           | 值                                |
  | -------------- | --------------------------------- |
  | **命令行格式** | `--basedir=dir_name`              |
  | **系统变量**   | `basedir`                         |
  | **范围**       | Global                            |
  | **动态**       | 不支持                            |
  | **类型**       | 目录名称                          |
  | **默认值**     | `configuration-dependent default` |

  MySQL的安装基础目录。

- `big_tables`

  | 属性           | 值              |
  | -------------- | --------------- |
  | **命令行格式** | `--big-tables`  |
  | **系统变量**   | `big_tables`    |
  | **范围**       | Global，Session |
  | **动态**       | 支持            |
  | **类型**       | 布尔            |
  | **默认值**     | `OFF`           |

  如果设置为1，则所有临时表都存储在磁盘而不是内存中。这会有些慢，但是对于需要大型临时表的`SELECT`操作不会发生`tbl_name` *is  full*的错误。新连接的默认值为0（使用内存）。通常，永远不需要设置此变量，因为内存表会根据需要自动转换为基于磁盘的表。

- `bind_address`

  | **命令行格式** | `--bind-address=addr` |
  | -------------- | --------------------- |
  | **系统变量**   | `bind_address`        |
  | **范围**       | Global                |
  | **动态**       | 不支持                |
  | **类型**       | 字符串                |
  | **默认值**     | `*`                   |

  `--bind-address`选项的值。MySQL服务器监听的IP地址。

- `block_encryption_mode`

  | 属性           | 值                          |
  | -------------- | --------------------------- |
  | **命令行格式** | `--block-encryption-mode=#` |
  | **系统变量**   | `block_encryption_mode`     |
  | **范围**       | Global，Session                  |
  | **动态**       | 支持                        |
  | **类型**       | 字符串                     |
  | **默认值**     | `aes-128-ecb`               |

  此变量控制基于区块的算法（如AES）的区块加密模式。它会影响`AES_ENCRYPT()`和`AES_DECRYPT`的加密。

  `block_encryption_mode`采用`aes-keylen`模式格式的值。其中`keylen`是以位为单位的密钥长度，`mode`是加密模式。该值不区分大小写。允许的`keylen`值有`128`,`192`,`256`。允许的加密模式取决于MySQL是使用`OpenSSL`还是`yaSSL`：

  - 对于`OpenSSL`，允许的模式值是：`ECB`, `CBC`, `CFB1`, `CFB8`, `CFB128`, `OFB`
  - 对于`yaSSL`，允许的模式值是：`ECB`, `CBC`

  例如，此语句使`AES`加密函数使用`256`为密钥长度和`CBC`模式：

  ```mysql
  SET block_encryption_mode = 'aes-256-cbc'；
  ```

- `bulk_insert_buffer_size`

  | 属性                   | 值                            |
  | ---------------------- | ----------------------------- |
  | **命令行格式**         | `--bulk-insert-buffer-size=#` |
  | **系统变量**           | `bulk_insert_buffer_size`     |
  | **范围**               | Global，Session               |
  | **动态**               | 支持                            |
  | **类型**               | 整数                          |
  | **默认值**             | `8388608`                     |
  | **最小值**             | `0`                           |
  | **最大值**（64位平台） | `18446744073709551615`        |
  | **最大值**（32位平台） | `4294967295`                  |

  在将数据添加到非空表时，MyISAM使用特殊的树状缓存来更快地为`INSERT ... SELECT`,`INSERT ... VALUES (...)`和`LOAD DATA`进行批量插入。此变量以每个线程的字节数限制缓存树的大小。将其设置为0将禁用此优化，默认值为8MB。

- `character_set_client`

  | 属性                | 值                  |
  | ------------------- | ---------------------- |
  | **系统变量**         | `character_set_client` |
  | **范围**           | Global, Session        |
  | **动态**            | 支持                    |
  | **类型**            | 字符串                 |
  | **默认值**   | `utf8`                 |

  客户端语句的字符集。当客户端连接到服务器时，使用客户单请求的字符集设置此变量的会话值。（许多客户端支持一个`--default-chaaracter-set`选项，可以明确指定此字符集。）变量的全局用于在客户端请求的情况下设置会话值不可用，或者服务器配置为忽略客户端请求：

  - 客户端请求服务器不知道的字符集。
  - 客户端来自早于MySQL4.1的版本。
  - mysqld是使用`--skip-character-set-client-handshake`选项启动的。

- `character_set_connection`

  | 属性         | 值                         |
  | ------------ | -------------------------- |
  | **系统变量** | `character_set_connection` |
  | **范围**     | Global, Session            |
  | **动态**     | 支持                       |
  | **类型**     | 字符串                     |
  | **默认值**   | `utf8`                     |

  用于文字的字符集，没有字符集导入器和数字到字符串转换。

- `character_set_database`

  | 属性         | 值                                                           |
  | ------------ | ------------------------------------------------------------ |
  | **系统变量** | `character_set_database`                                     |
  | **范围**     | Global, Session                                              |
  | **动态**     | Yes                                                          |
  | **类型**     | 字符串                                                       |
  | **默认值**   | `latin1`                                                     |
  | **备注**     | 此选项是动态的，但只有服务器应设置此信息。您不应手动设置此变量的值 |

  默认数据库使用的字符集。每当默认数据库库更改时，服务器都会设置此变量。如果没有默认数据库。则该变量具有相同的值。

  变量在以后的版本弃用。

  5.7版本中不推荐为此变量赋值。对此变量赋值会产生警告。

- `character_set_filesystem`

  | 属性           | 值                                |
  | -------------- | --------------------------------- |
  | **命令行格式** | `--character-set-filesystem=name` |
  | **系统变量**   | `character_set_filesystem`        |
  | **范围**       | Global, Session                   |
  | **动态**       | 支持                               |
  | **类型**       | 字符串                            |
  | **默认值**     | `binary`                            |

  文件系统字符集。次变量用于解释引用文件名的字符串文字。例如在`LOAD DATA`和`SELECT ... INTO OUTFILE`语句和 `LOAD_FILE()`函数中。这样的文件名从转换`character_set_client`到`character_set_filesystem`发生文件打开尝试之前。默认值为`binary`表示不进行转换。对于允许使用多字节文件名的系统，不同的值可能更适合。例如，如果系统使用UTF-8，则设置`character_set_filesystem`为`utf8mb4`。

- `character_set_results`

  | 属性         | Value                   |
  | ------------ | ----------------------- |
  | **系统变量** | `character_set_results` |
  | **范围**     | Global, Session         |
  | **动态**     | 支持                    |
  | **类型**     | 字符串                  |
  | **默认值**   | `utf8`                  |

  用于将查询结果返回给客户端的字符集。这包括结果数据，如列值，结果元数据和错误消息。

- `character_set_server`

  | Property       | Value                    |
  | -------------- | ------------------------ |
  | **命令行格式** | `--character-set-server` |
  | **系统变量**   | `character_set_server`   |
  | **范围**       | Global, Session          |
  | **动态**       | Yes                      |
  | **类型**       | String                   |
  | **默认值**     | `latin1`                 |

  服务器的默认字符集。

- `character_set_system`

  | 属性              | 值                     |
  | ----------------- | ---------------------- |
  | **系统变量**      | `character_set_system` |
  | **范围**          | Global                 |
  | **动态**          | 不支持                 |
  | **类型**          | 字符串                 |
  | **Default Value** | `utf8`                 |

  服务器用于存储标识符的字符集。默认值为 `utf8`.

- `character_sets_dir`

  | 属性           | 值                              |
  | -------------- | ------------------------------- |
  | **命令行格式** | `--character-sets-dir=dir_name` |
  | **系统变量**   | `character_sets_dir`            |
  | **范围**       | Global                          |
  | **动态**       | No                              |
  | **类型**       | 目录名                          |

  字符集的目录

- `check_proxy_users`

  | 属性           | 值                                |
  | -------------- | --------------------------------- |
  | **命令行格式** | `--check-proxy-users=[={OFF|ON}]` |
  | **系统变量**   | `check_proxy_users`               |
  | **范围**       | Global                            |
  | **动态**       | Yes                               |
  | **类型**       | 布尔                              |
  | **默认值**     | `OFF`                             |

  一些身份验证插件为自己实现代理用户映射（例如，PAM和Winodws身份验证插件）。默认情况下，其他身份验证插件不支持代理用户。其中，有些可以要求MySQL扶我去器本身根据授予的代理权限映射给代理用户：`mysql_natice_password`,`sha256_password`。

  如果启用了`check_proxy_users`系统变量，则服务器会对发出此请求的任何身份验证插件执行代理用户映射。但是，可能还需要启用特定于插件的系统变量以利用服务器代理用户映射支持：

  - 对于`mysql_native_password`插件。启用

    `mysql_native_password_proxy_users`

  - 对于`sha256_password`插件。启用：

    `sha256_password_proxy_users`

  

- `collation_connection`

  | 属性         | 值                     |
  | ------------ | ---------------------- |
  | **系统变量** | `collation_connection` |
  | **范围**     | Global, Session        |
  | **动态**     | Yes                    |
  | **类型**     | 字符串                 |

  连接字符集的排序规则。对于文字字符串比的比较很重要。对于具有列值的字符串的比较无关紧要，因为列具有自己的排序规则，具有更改的排序规则优先级。

- `collation_database`

  | 属性         | 值                                                    |
  | ------------ | ----------------------------------------------------- |
  | **系统变量** | `collation_database`                                  |
  | **范围**     | Global, Session                                       |
  | **动态**     | Yes                                                   |
  | **类型**     | 字符串                                                |
  | **默认值**   | `latin1_swedish_ci`                                   |
  | **注**       | 选项使动态的，但只有服务器应设置此信息。不应手动设置. |

  默认数据库使用的排序规则。每当默认数据库更改时，服务器都会设置此变量。如果没有默认数据库，则该变量具有相同的值

- `collation_server`

  | 属性           | 值                   |
  | -------------- | -------------------- |
  | **命令行格式** | `--collation-server` |
  | **系统变量**   | `collation_server`   |
  | **范围**       | Global, Session      |
  | **动态**       | Yes                  |
  | **类型**       | String               |
  | **默认值**     | `latin1_swedish_ci`  |

  服务器的默认排序规则。.

- `completion_type`

  | 属性           | 值                                       |
  | -------------- | ---------------------------------------- |
  | **命令行格式** | `--completion-type=#`                    |
  | **系统变量**   | `completion_type`                        |
  | **范围**       | Global, Session                          |
  | **动态**       | Yes                                      |
  | **类型**       | Enumeration                              |
  | **默认值**     | `NO_CHAIN`                               |
  | **有效值**     | `NO_CHAIN` `CHAIN` `RELEASE` `0` `1` `2` |

  事务完成类型。此变量可以采用下列值。可使用名称值或相应的整数值

  | 值               | 描述                                                         |
  | ---------------- | ------------------------------------------------------------ |
  | `NO_CHAIN`(or 0) | `COMMIT`和 `ROLLBACK`不受影响。这是默认值。                  |
  | `CHAIN` (or 1)   | `COMMIT`和`ROLLBACK` 分别等同于 `COMMIT AND CHAIN` 和`ROLLBACK AND CHAIN`, 新事务立即启动，其隔离级别与刚刚终止的事务相同。 |
  | `RELEASE`(or 2)  | `COMMIT`并且 `ROLLBACK` 分别等同于`COMMIT RELEASE`和 `ROLLBACK RELEASE`。（终止事务后服务器断开连接。） |

  

- `concurrent_insert`

  | 属性           | 值                             |
  | -------------- | ------------------------------ |
  | **命令行格式** | `--concurrent-insert[=#]`      |
  | **系统变量**   | `concurrent_insert`            |
  | **范围**       | Global                         |
  | **动态**       | 支持                           |
  | **类型**       | Enumeration                    |
  | **默认值**     | `AUTO`                         |
  | **有效值**     | `NEVER``AUTO``ALWAYS``0``1``2` |

  如果设置为AUTO（默认值），MySQL允许`INSERT`和`SELECT`语句同事运行MyISAM在数据文件中间没有空闲块的表。如果启动`mysqld`时使用`–skip-new`，这个变量被设置为NEVER。

  

- `connect_timeout`

  | 属性           | 值                    |
  | -------------- | --------------------- |
  | **命令行格式** | `--connect-timeout=#` |
  | **系统变量**   | `connect_timeout`     |
  | **范围**       | Global                |
  | **动态**       | Yes                   |
  | **类型**       | Integer               |
  | **默认值**     | `10`                  |
  | **最小值**     | `2`                   |
  | **最大值**     | `31536000`            |

  `mysqld`服务器在响应`Bad handshake`之前等待连接数据包的秒数。默认为10s

- `core_file`

  | 属性         | 值          |
  | ------------ | ----------- |
  | **系统变量** | `core_file` |
  | **范围**     | Global      |
  | **动态**     | No          |
  | **类型**     | Boolean     |
  | **默认值**   | `OFF`       |

  是否在服务器崩溃时写入核心文件。由 `--core-file` 选项设置。

- `datadir`

  | 属性           | 值                   |
  | -------------- | -------------------- |
  | **命令行格式** | `--datadir=dir_name` |
  | **系统变量**   | `datadir`            |
  | **范围**       | Global               |
  | **动态**       | 不支持               |
  | **类型**       | 目录名               |

  MySQL服务器数据目录的路径，相对于当前目录解析相对路径。如果服务器将自动启动（无法假设当前目录的向下文），最好将`datadir`值指定为绝对路径。

- `date_format`

  未使用，将被弃用。

- `datetime_format`

  未使用，将被弃用。

- `debug`

  | 属性                  | 值                          |
  | --------------------- | --------------------------- |
  | **命令行格式**        | `--debug[=debug_options]`   |
  | **系统变量**          | `debug`                     |
  | **范围**              | Global，Session             |
  | **动态**              | 支持                        |
  | **类型**              | 字符串                      |
  | **默认值**（Windows） | `d:t:i:O,\mysqld.trace`     |
  | **默认值**（Unix）    | `d:t:i:o,/tmp/mysqld.trace` |

  此变量指定当前的`debug`设置。既适用于使用调试支持构建的服务器。初始值来自服务器启动时给出的`debug`选项的实例。可以在运行时设置全局和会话值。

  分配以`+`或`-`开头的值会导致将值添加到当前或从当前减去该值：

  ```mysql
  SET debug = 'T';
  SELECT @@debug;
  +---------+
  | @@debug |
  +---------+
  | T       |
  +---------+
  SET debug = '+P';
  SELECT @@debug;
  +---------+
  | @@debug |
  +---------+
  | P:T     |
  +---------+
  
  SET debug = '-P';
  SELECT @@debug;
  +---------+
  | @@debug |
  +---------+
  | T       |
  +---------+
  ```

- `default_authentication_plugin`

  | 属性           | 值                                            |
  | -------------- | --------------------------------------------- |
  | **命令行格式** | `--default-authentication-plugin=plugin_name` |
  | **系统变量**   | `default_authentication_plugin`               |
  | **范围**       | Global                                          |
  | **动态**       | 不支持                                          |
  | **类型**       | Enumeration                                   |
  | **默认值**     | `mysql_native_password`                       |
  | **有效值**     | `mysql_native_password``sha256_password`      |

  默认的身份验证插件。允许使用这些：

  - `mysql_native_password`:使用本地MySQL密码；
  - `sha256_password`：使用`SHA-256`密码；

  > 如果此变量的值不是`mysql_native_password`，则5.5.7之前版本的客户端无法连接。

  `default_authentication_plugin`的值会影响这些方面：

  - 它确定服务器为`CREATE USER`创建的新账户分配哪个身份验证插件，以及未明确指定身份验证插件的`GRANT`语句。

  - `old_passwords`系统变量影响使用`mysql_native_password`或`sha256_password`身份验证插件的账户的密码哈希。如果默认身份验证插件是其中一个。则服务器会在启动时将`old_passwords`设置为插件密码哈希方法所需的值。

  - 对于使用以下任一语句创建的账户，服务器将该账户与默认身份验证插件关联，并为该账户分配密码，根据插件的要求进行哈希处理。 

    ```mysql
    CREATE USER ... IDENTIFIED BY 'cleartext password';
    GRANT ... IDENTIFIED BY 'cleartext password';
    ```

  - 对于使用以下任一语句创建的用户，如果密码哈希具有插件所需的格式，则服务器会将该账户与默认身份验证插件相关联，并为该账户分配给定的密码哈希值。

    ```mysql
    CREATE USER ... IDENTIFIED BY PASSWORD 'encrypted password';
    GRANT ... IDENTIFIED BY PASSWORD 'encrypted password';
    ```

    如果密码哈希值不是默认身份验证插件所需的格式，则该语句失败。

  - `default_password_life_time`

    | 属性                     | 值                              |
    | ------------------------ | ------------------------------- |
    | **命令行格式**           | `--default-password-lifetime=#` |
    | **系统变量**             | `default_password_lifetime`     |
    | **范围**                 | Global                            |
    | **动态**                 | 支持                              |
    | **类型**                 | 整数                            |
    | **默认值**（> = 5.7.11） | `0`                             |
    | **默认值**（<= 5.7.10）  | `360`                           |
    | **最小值**           | `0`                             |
    | **最大值**             | `65535`                         |

    此变量定义全局自动密码到期策略。默认值为`0`，禁用密码自动到期。如果值为正整数N，则表示允许的密码生存期；密码必须每N天更改一次。

    可以使用`ALTER USER`语句的密码到期选项根据需要覆盖全局密码到期策略。

    > 从MySQL 5.7.4到5.7.10，默认`default_password_lifetime` 值为360（密码每年大约必须更改一次）。对于这些版本，请注意，如果您不对`default_password_lifetime` 变量或单个用户帐户进行任何更改 ，则所有用户密码将在360天后过期，并且所有用户帐户将在发生这种情况时开始以受限模式运行。连接到服务器的客户端（实际上是用户）将收到错误，指示必须更改密码： `ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`
    >
    > 但是，对于自动连接到服务器的客户端（例如通过脚本建立的连接），很容易错过。为避免此类客户端因密码过期而突然停止工作，请确保更改这些客户端的密码过期设置，如下所示：
    >
    > ```mysql
    > ALTER USER 'script'@'localhost' PASSWORD EXPIRE NEVER
    > ```
    >
    > 或者，将`default_password_lifetime` 变量设置 为`0`，从而禁用所有用户的自动密码到期。

  - `default_storage_engine`

    | 属性           | 值                              |
    | -------------- | ------------------------------- |
    | **命令行格式** | `--default-storage-engine=name` |
    | **系统变量**   | `default_storage_engine`        |
    | **范围**       | 全球，会议                      |
    | **动态**       | 支持                            |
    | **类型**       | Enumeration                     |
    | **默认值**     | `InnoDB`                        |

    默认存储引擎，此变量仅为永久表设置存储引擎，要为`TEMPORARY`表设置存储引擎，请设置`default_tmp_storage_engine`系统变量。

    要查看哪些存储引擎可用并以启用，使用`SHOW ENGINES`语句或查询`INFORMATION_SCHEMA`的`ENGINES`表。

    `default_storage_engine`应优先于`storage_engine`使用，不推荐，并在5.7.5中删除。

    如果服务器启动时禁用某些存储引擎，则必须将`permanent`和`TEMPORARY`表的默认引擎设置为其他引擎，否则服务器将无法启动。

