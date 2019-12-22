# Ansible命令行工具

**针对一组主机定义并运行单个任务剧本**

概要

```bash
usage: ansible [-h] [--version] [-v] [-b] [--become-method BECOME_METHOD]
            [--become-user BECOME_USER] [-K] [-i INVENTORY] [--list-hosts]
            [-l SUBSET] [-P POLL_INTERVAL] [-B SECONDS] [-o] [-t TREE] [-k]
            [--private-key PRIVATE_KEY_FILE] [-u REMOTE_USER]
            [-c CONNECTION] [-T TIMEOUT]
            [--ssh-common-args SSH_COMMON_ARGS]
            [--sftp-extra-args SFTP_EXTRA_ARGS]
            [--scp-extra-args SCP_EXTRA_ARGS]
            [--ssh-extra-args SSH_EXTRA_ARGS] [-C] [--syntax-check] [-D]
            [-e EXTRA_VARS] [--vault-id VAULT_IDS]
            [--ask-vault-pass | --vault-password-file VAULT_PASSWORD_FILES]
            [-f FORKS] [-M MODULE_PATH] [--playbook-dir BASEDIR]
            [-a MODULE_ARGS] [-m MODULE_NAME]
            pattern
```

## 描述

用于执行“执行操作”的简单工具/框架/API。该命令允许针对一组主机定义并运行单个任务剧本。

## 常用选项

>  --ask-vault-pass

要求提供`vault`密码

>  --become-method <BECOM_METHOD>

要使用的特权方法 ，使用`ansible-doc -t become -l `列出可用选项

> --become-user <BECOME_USER>

以这个用户的身份运行操作，默认root

> --list-hosts

输出匹配的主机列表；不执行其他任何操作

> --playbook-dir <BASEDIR>

由于此工具不使用剧本，因此可以将其用作替代剧本目录，从而为许多功能设置相对路径，包括`roles/group_vars`。

> --private-key <PRIVATE_KEY_FILE>, --key-file <PRIVATE_KEY_FILE>

使用此文件来验证连接

> --scp-extra-args <SCP_EXTRA_ARGS>

指定额外的参数以仅传递给scp（例如-l）

> --sftp-extra-args <SFTP_EXTRA_ARGS>

指定额外的参数以仅传递给sftp（例如-f，-l）

> --ssh-common-args <SSH_COMMON_ARGS>

指定要传递给sftp/scp/ssh的通用参数（例如ProxyCommand）

>  --ssh-extra-args <SSH_EXTRA_ARGS>

指定额外的参数以仅传递给ssh（例如-R）

> --syntax-check

在剧本上执行语法检查，但不执行

> --vault-id

要使用的vault身份

> --vault-password-file

vault密码文件

> --version

显示程序的版本号，配置文件位置，配置的模块搜索路径，模块位置，可执行文件位置和退出

> -B <SECONDS>, --background <SECONDS>

异步运行，在X秒后失败（默认值= N/A）

> -C, --check

不要做任何改变；相反，尝试预测可能发生的某些变化

> -D, --diff

更改（小的）文件和模板时，请显示这些文件中的差异；与–check一起使用效果很好

> -K, --ask-become-pass

要求特权升级密码

> -M, --module-path

将冒号分隔的路径添加到模块库（默认=~/ .ansible/plugins/modules:/usr/share/ansible/plugins/modules）

> -P <POLL_INTERVAL>, --poll <POLL_INTERVAL>

如果使用-B，则设置轮询间隔（默认值=15）

> -T <TIMEOUT>, --timeout <TIMEOUT>

覆盖连接超时（以秒为单位）（默认为10）

> -a <MODULE_ARGS>, --args <MODULE_ARGS>

模块参数

> -b, --become

使用变为运行操作（不表示提示输入密码）

> -c <CONNECTION>, --connection <CONNECTION>

要使用的连接类型（默认=smart）

> -e, --extra-vars

如果文件名以@开头，则将其他变量设置为key=value或YAML/JSON

> -f <FORKS>, --forks <FORKS>

指定要使用的并行进程数（默认=5）

> -h, --help

显示此帮助消息并退出

> -i, --inventory, --inventory-file

指定清单主机路径或逗号分隔的主机列表。–不推荐使用库存文件

> -k, --ask-pass

询问连接密码

> -l <SUBSET>, --limit <SUBSET>

将所选主机进一步限制为其他模式

> -m <MODULE_NAME>, --module-name <MODULE_NAME>

要执行的模块名称（默认=命令）

> -o, --one-line

压缩输出

> -t <TREE>, --tree <TREE>

日志输出到该目录

> -u <REMOTE_USER>, --user <REMOTE_USER>

以该用户身份连接（默认=None）

> -v, --verbose

详细模式（-vvv用于更多，-vvvv用于启用连接调试）



## Environment

可以用来指定以下环境变量。

`ANSIBLE_CONFIG` - 覆盖默认的ANSIBLE配置文件

ansible.cfg中大多数选项都可以使用

## 文件

`/etc/ansible/ansible.cfg` - 配置文件（如果存在）

`~/.ansible.cfg`  - 用户配置文件，覆盖默认配置（如果存在）

