# Ansible临时命令

Ansible ad-hoc命令使用`/usr/bin/ansible`命令行工具来自动化一个或多个受管节点上的单个任务。临时命令既快速有简单，但不可重复使用。为什么首先要了解临时命令，临时命令展示了Ansible的简单性和强大功能。在此处学习的概念将直接移植到playbook。



## 为什么要使用临时命令？

临时命令非常适合很少重复性的任务。例如，如果想要在圣诞节假期关闭实验室中的所有机器的电源，则可以在Ansible中执行快速的一次工作而不需编写剧本。临时命令如下所示：

```bash
ansible [pattern] -m [module] -a "[module opetions]"
```

## 临时任务用例

临时任务可用于重新引导服务器,复制文件,管理程序包和用户等。可以在临时任务中使用任何Ansible模块。临时任务使用声明性模型，计算并执行达到指定最终状态所需的操作。通过在开始之前检查当前状态并且不执行任何操作，除非当前状态与指定的最终状态不同，它们可以实现幂等形式。

### 重新启动服务器

`ansible`命令实用程序的默认模块是`command module`。可以使用临时任务来调用`command`模块,然后依次重启所有Web服务器.每次十个。在Ansible执行此操作前，必须在清单中一个名为`[atlanta]`的组中列出所有服务器,并且该组中的媚态计算机都必须具有有效的SSH凭据。要重新启动组中的所有服务器:

```baah
ansible atlanta -a "/sbin/reboot"
```

默认情况下,Ansible仅使用5个并发进程.如果拥有主机数量超过为派生计数设置的值,会花费一些时间。要使用10个并发分支重新启动[atlanta]服务器组,请执行以下操作:

```bash
ansible atlanta -a "/sbin/reboot" -f 10 
```

`/usr/bin/ansible`将默认从您的用户账户运行.要以其他我身份连接:

```bash
ansible atlanta -a "/sbin/reboot" -f 10 -u username
```

重新引导可能需要特权升级.可以使用以下关键字变为用户身份连接到服务器,并使用`become`关键字作为`root`用户运行该命令。

```bash
ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]
```

如果添加`--ask-become-pass`或`-K`，则Ansible会提示输入用于特权提升的密码

到目前位置，我们所有的示例都默认使用`command`模块。要使用其他模块，请为`-m`参数传递模块名称。例如，使用`shell`模块:

```bash
ansible centos -m shell -a 'echo $TERM'
```

当使用 Ansible ad hoc 命令运行任何命令时，请特别注意shell引用规则，因此本地shell会保留该变量并将其传递给Ansible。例如，在上面的示例中使用双引号而不是单引号将对变量进行求值。

### 管理文件

临时任务可以利用Ansible和SCP的功能将许多文件并行传输到多台计算机。要将文件直接传输到[centos]组中的所有服务器,请执行以下操作:

```bash
ansible centos -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```

如果打算重复这样的任务,请在playbook中使用`template`模块。

`file`模块允许改变文件所有权和权限。这些相同的选项也可以传递给`copy`模块:

```bash
ansible centos -m file -a "dest=/tmp/hosts mode=600" 
ansible centos -m file -a "dest=/tmp/hosts mode=600 owner=root gourp=root" 
```

`file`模块还可以创建目录,类似`mkdir -p`:

```bash
ansible centos -m file -a "dest=/tmp/12213 mode=755 owner=test1 group=test1 state=directory"
```

以及递归删除目录和文件:

```bash
ansible centos -m file -a "dest=/tmp/12213 state=absent"
```

### 管理包

您还可以使用临时任务来使用软件包管理模块(例如yum)在受管节点上安装,更新或删除软件包。要确保仅安装而不更新它：

```bash
ansible centos -m yum -a "name=acme state=present"
```

要确保安装软件特定版本

```bash
ansible centos -m yum -a "name=acme-1.5 state=present"
```

要确保软件包为最新版:

```bash
ansible centos -m yum -a "name=acme state=latest"
```

确保未安装软件包:

```bash
ansible centos -m yum -a "name=acme state=absent"
```

Ansible具有用于在许多平台下管理软件包的模块。如果您的软件包管理器没有模块，则可以使用命令模块安装软件包或为软件包管理器创建模块。

### 管理用户和组

