# Ansible 安装指南

## 基础知识

默认情况下,Ansible通过`SSH`协议管理计算机。

一旦安装了Ansible，不会添加数据库，也没有启动或继续运行的守护程序。只需要将其安装在一台计算机上，它就可以从该中心管理远程计算机。当Ansible管理远程计算机时，它不会在其上安装或运行软件，因此，升级Ansible不会影响任何问题。



## 选用哪个版本？

由于Ansible可以从源代码轻松运行，并且不需要再远程计算机上安装任何软件，因此许多用户实际上会跟踪开发版本。

如果希望运行最新版本的Ansible，并且运行在发行版上，则建议使用OS软件包管理器

对于其他安装选项，建议通过Python的`pip`软件包管理器安装。

## 控制节点要求

当前,Ansible可以在装有`Python2`或`Python3`的任何计算机上运行，但可惜的是，Ansible的控制节点不支持Windows。

## 受控节点要求

受控节点上，需要一种通信方式，通常是`SSH`。默认情况下，使用`sftp`。如果不可用，可以在`ansible.cfg`切换到`scp`。当然，还需要`Python2`或`Python3`。

> 注意：
>
> - 如果在远端节点上启用了`SELinux`，则还需要在Ansible中使用任何与`copy`、`file`、template相关功能之前，在它们上安装`libselinux-python`。
>
> - 默认情况下，Ansible使用位于`/usr/bin/python`的python解释器运行其模块。但是，但是，默认情况下，某些Linux发行版仅将Python3解释器安装到`/usr/bin/python3`。在这些系统上，可能会遇到如下错误：
>
>   ```python
>   "module_stdout": "/bin/sh: /usr/bin/python: No such file or directory\r\n"
>   ```
>
>   可以将`ansible_python_interpreter`inventory变量设置为指定解析器,或者可以安装`Python2`解释器以供模块使用。如果未将`Python2`解释器安装到`/usr/bin/python`,则仍需设置`ansible_python_interpreter`。
>
> - Ansible的`raw`模块和脚本模块不依赖于客户端的Python来运行。从技术上来讲，可以使用Ansible `raw`模块安装兼容的Python，然后使用该模块使用其他所有模块。如果需要将Python2安装到基于RHEL的系统上，则可以通过以下方式安装：
>
>   ```bash
>   ansible myhost --become -m raw -a "yum install -y python2"
>   ```



## 安装控制节点

*这里仅以CentOS举例,其他系统请参考文档.*

### 在RHEL和CentOS上:

```bash
sudo yum install ansible 
```

要为RHEL8启用Ansible Engine存储库,需要运行如下命令:

```bash
sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
```

要为RHEL7启用Ansible Engine存储库,需要运行如下命令:

```bash
sudo subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms
```

当前可支持的RHEL，CentOS和Fedora版本的RPM可从[EPEL](https://fedoraproject.org/wiki/EPEL)以及[releases.ansible.com获得](https://releases.ansible.com/ansible/rpm)。

Ansible 2.4和更高版本可以管理包含Python 2.6或更高版本的早期操作系统。

### 通过Pip

可以通过Python的`pip`软件包管理器安装Ansible。如果`pip`尚不可用,请运行以下命令进行安装:

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
```

然后安装Ansible:

```bash
pip install --user ansible
```

如果要全局安装Ansible,运行如下命令:

```bash
sudo python git-pip.py
sudo pip install ansible
```

## Shell Completion

Ansible2.9开始，ansible命令行实用程序的`shell completion`是可用的,通过一个可选的`argcomplete`提供。`argcomplete`支持`bash`,但对`fish`和`zsh`支持有限。

### 安装

*这里以CentOS为例*

```bash
yum install epel-release -y 
yum install python-argcomplete -y 
```

### 配置

有两种方法配置`argcomplete`

#### 全局

*全局需要bash4.2*

```bash
sudo activate-global-python-argcomplete
```

#### 每个命令

如果没有bash4.2。则必须独立注册每个脚本。

```bash
eval $(register-python-argcomplete ansible)
eval $(register-python-argcomplete ansible-config)
eval $(register-python-argcomplete ansible-console)
eval $(register-python-argcomplete ansible-doc)
eval $(register-python-argcomplete ansible-galaxy)
eval $(register-python-argcomplete ansible-inventory)
eval $(register-python-argcomplete ansible-playbook)
eval $(register-python-argcomplete ansible-pull)
eval $(register-python-argcomplete ansible-vault)
```

建议将上述命令放入shell概要文件中,例如`~/.profile`或`~/.bash_profile`。

