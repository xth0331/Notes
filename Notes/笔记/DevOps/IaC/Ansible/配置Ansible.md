# Ansible 配置



## 配置文件

Ansible中的某些设置可以通过配置文件`ansible.cfg`进行调整。

### 获取最新的配置

如果从程序包管理器安装Ansible，则最新的`ansible.cfg`文件应该位于`/etc/ansible`中，兵可能在更新时作为`.rpmnew`存在。

如果从`pip`或从源码安装,则可能要创建此文件以覆盖`ansible.cfg`的默认设置。

## 环境配置

Ansible还允许使用环境变量配置设置。如果设置了这些环境变量，将覆盖从配置文件加载的所有配置。

## 命令行选项

并非命令行中存在所有配置选项，只有最有用或者最常用的配置选项。命令行中的设置将覆盖通过配置该文件和环境传递的设置。

>配置文件参考:
>
>https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings
>
>https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg
>
>







