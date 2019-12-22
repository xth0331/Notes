# ansible-config命令行工具

*查看ansible配置*



## 概要

```bash
usage: ansible-config [-h] [--version] [-v] {list,dump,view} ...
```

## 常用选项

> --version

显示程序版本号，配置文件位置，配置的模块搜索路径，模块位置，可执行文件位置

> -h --help

显示帮助

> -v，--verbose

详细模式（-vvv用于更多，-vvvv用户启用连接调试）



## 动作

### list

列出所有当前读取`lib/constants.py`的配置，并显示环境和配置文件设置名称

> -c <CONFIG_FILE>, --config <CONFIG_FILE>

配置文件的路径，默认为优先找到的第一个文件。

### dump

显示当前设置，合并`ansible.cfg`（如果指定）

> --only-changed

仅显示从默认更改的配置

> -c <CONFIG_FILE>, --config <CONFIG_FILE>

配置文件的路径，默认为优先找到的第一个文件。

### view

显示当前配置文件

> -c <CONFIG_FILE>, --config <CONFIG_FILE>

配置文件的路径，默认为优先找到的第一个文件。

## Environment

可以指定以下环境变量。

ANSIBLE_CONFIG - 覆盖默认的ANSIBLE配置文件。

`ansible.cfg`中大多数选项都可以使用。

## 文件

`/etc/ansible/ansible.cfg` – 配置文件，如果存在

`~/.ansible.cfg` – 用户配置文件，覆盖默认配置文件（如果存在）