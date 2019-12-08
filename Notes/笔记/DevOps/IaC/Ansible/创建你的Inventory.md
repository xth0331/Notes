# 如何建立你inventory

Ansible使用成为清单的一个列表或一组列表,同事针对基础架构中的多个受管节点或主机进行工作。定义清单后，可以使用模式来选择要运行Ansible的主机或组。

Inventory的默认位置是`/etc/ansible/hosts`。可以使用`-i`选项在命令行中指定其他清单文件。可以同时使用多个清单文件,从动态或云资源或不同格式(YAML,ini等)中提取清单。

## 清单基础知识：格式，主机和组

清单文件可以采用多种格式，具体取决拥有的清单插件。最常用的格式是`ini`和`YAML`。基本的`INI`像`/etc/ansible/hosts`可能看起来像这样。

```ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

括号中的标题是组名,用于对主机进行分类并确定在什么时候,什么目的控制什么主机。

这里是一个相同的基本清单文件，使用`YAML`格式:

```yaml
all:
  hosts:
    mail.example.com
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        tow.example.com:
        three.example.com:
```

### 默认组

有两个默认组: `all`和`ungrouped`。`all`组包含每个主机。`ungrouped`组包含除所有主机之外没有其他组的所有主机。每个主机都至少属于两个组(`all`和`ungroups`或`all`和其他某个组)，尽管`all`和`ungrouped`始终存在,但它们是可以隐式的,不会出现在`group`列表中。

### 主机在多个组

可以将每个主机分成多个组。例如，数据中心中的生产Web服务器可能包含在名为`[prod]`和`[atlanta]`和`[webservers]`的组中

- What - 应用程序、堆栈或微服务。例如，数据库服务器、web服务器等。
- Where - 一个数据中心或区域，与本地DNS、存储器等通信（例如，东，西）。
- When - 在开发阶段,避免对生产资源进行测试。（例如生产，测试）。

扩展当前的YAML清单，以包括What，When，Where：

```yaml
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
    east:
      hosts:
        foo.example.com:
        one.example.com:
        two.example.com:
    west:
      hosts:
        bar.example.com:
        three.example.com:
    prod:
      hosts:
        foo.example.com:
        one.example.com:
        two.example.com:
    test:
      hosts:
        bar.example.com:
        three.example.com:
```

可以看到`one.example.com`存在`dbservers`，`east`和`prod`组。

对于相同的结果，还可以使用嵌套的组来简化这个目录中的`prod`和`test`:

```yaml
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
    east:
      hosts:
        foo.example.com:
        one.example.com:
        two.example.com:
    west:
      hosts:
        bar.example.com:
        three.example.com:
    prod:
      children:
        east:
    test:
      children:
        west:
```



### 添加主机范围

如果有很多具有类似模式的主机，则可以将它们添加为范围，而不必分别列出每个主机名：

`INI`:

```ini
[webservers]
www[01:50].example.com
```

`YAML`:

```yaml
  webservers:
    hosts:
      www[01:50].example.com:
```

还可以定义字母范围:

```ini
[databases]
db-[a:f].example.com
```

 ## 向清单添加变量

 可以存储于清单中特定主机或组相关的变量值.首先,可以将变量直接添加到主清单文件的主机和组.但是,随着将越来越多受管节点添加到Ansible清单中,可能希望将变量存储在单独的主机和组变量文件中。

## 将变量分配给一台主机： 主机变量

可以轻松地将变量分配给单个主机,然后在剧本中使用它。在`INI`中:

```ini
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

`YAML`:

```yaml
atlanta:
  host1:
    http_port: 80
    maxRequestsPerChild: 808
  host2:
    http_port: 303
    maxRequestsPerChild: 909
```

诸如非标准SSH端口的唯一值可以很好地用作主机变量.可以通过在主机名后加冒号的端口号将它们添加到Ansible清单中:

```ini
badwolf.example.com:5309
```

连接变量也可以用作主机变量:

```ini
[targets]
localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```

> 注意
>
> 如果在SSH配置文件中列出了非标准SSH端口,则`openssh`连接将找到并使用它们,但`paramiko`连接将找不到。

### 清单别名

可以在清单中定义别名:

`INI`:

```ini
jumper ansible_port=5555 ansible_host=192.0.2.50
```

`YAML`:

```yaml
...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```

在上面的示例中,对主机别名`jumper`运行Ansible将连接端口`192.0.2.50`的5555端口上.这仅适用具有静态IP的主机,或者通过隧道连接时。

通常，者不是定义描述系统策略的变量的最佳方法。在主清单文件中设置变量只是一种简写。

## 将变量分配给许多主机： 组变量

如果租中的所有主机共享一个变量值，则可以一次性将该变量应用于整个组。

`INI`:

```ini
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

`YAML`:

```yaml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```

组变量是一次将变量应用于多个主机的便捷方法.但是,在执行之前,Ansible使用将变量展开到主机级别。如果主机是多个组的成员，则Ansible将从所有这些组中读取变量值。如果将不同的值分配给不同组中的同一变量，则Ansib将根据内部合并规则选择要使用的值。

### 继承变量值: 组的组的组变量

可以使用`ini`中的`:children`后缀或`yaml`中的`children:`条目进行分组,可以使用:`vars`或`vars:`将变量应用于这些组:

`INI`:

```ini
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

`YAML`:

```yaml
all:
  children:
    usa:
      children:
        southeast:
          children:
            atlanta:
              hosts:
                host1:
                host2:
            raleigh:
              hosts:
                host2:
                host3:
          vars:
            some_server: foo.southeast.example.com
            halon_system_timeout: 30
            self_destruct_countdown: 60
            escape_pods: 2
        northeast:
        northwest:
        southwest:
```

子组有几个要注意的属性:

- 属于子组成员的任何主机都自动成为父组成员
- 子组的变量将具有较高的优先级,覆盖父组的变量
- 组可以有多个父组和子组,但不能有循环关系
- 主机也可以在多个组,但只会有一个主机的情况下,合并来自多个组的数据

## 组织主机和组变量

Ansible通过搜索相对于清单文件或剧本文件的路径来加载主机和组变量文件。如果清单文件`/etc/ansible/hosts`中包含明文`foosball`主机,该主机属于`raleigh`和`webservers`两个组,则该主机将在以下位置的YAML文件中使用变量:

```bash
/etc/ansible/group_vars/raleigh 
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

例如,如果按数据中心对清单中的主机进行分组,并且每个数据中心使用其自己的NTP服务器和数据库服务器,则可以创建一个名为`/etc/ansible/group_vars/raleigh`来存储该`raleigh`组的变量:

```yaml
---
ntp_server: acme.example.org
database_server: storage.example.org
```

还可以创建以组或主机命名的目录.Ansible将按字典顺序读取这些目录中的所有文件,`raleigh`组的示例:

```bash
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```

组中所有主机都将具有在这些文件中定义的变量。当单个文件太大时，或者要对某些组变量使用`Ansible Vault`时,非常有用.

还可以将`group_vars/`和`host_vars/`目录添加到剧本目录中,默认情况下,`ansible-playbook`命令在当前工作目录中查找这些目录.其他`ansible`命令(例如`ansible`、`ansible-console`等)仅将在清单目录中查找`group_vars/`和`host_vars/`。如果要其他命令从剧本目录加载组和主机变量，则必须在命令行上提供`--playbook-dir`选项.如果同时从`playbook`目录和清单目录中加载文件,则`playbook`目录中的变量将覆盖在清单目录中设置的变量。

将清单文件和变量保存在`git repo`(或其他版本控制)中,是绝佳办法。

## 变量如何合并



默认情况下，在运行播放之前，变量将合并/展平到特定主机。这使Ansible始终专注于主机和任务，因此组无法真正在存货和主机匹配之外生存。默认情况下，Ansible会覆盖变量，包括为组和/或主机定义的变量。顺序/优先顺序是（从最低到最高）：

- 所有组（因为它是所有其他组的“父组”）
- 父组
- 子组
- host

默认情况下，Ansible按字母顺序合并处于相同父/子级别的组,最后加载的组将覆盖先前的组。例如，a_group将与b_group合并,并且匹配的b_group变量将覆盖a_group中的变量。



可以通过设置组变量 `ansible_group_priority` 来更改同级别的组的合并顺序。数字越大，合并的时间越晚，优先级更高。 如果未设置，则变量默认为`1`,例如:

```yaml
a_group:
    testvar: a
    ansible_group_priority: 10
b_group:
    testvar: b
```

在本例中,如果两个组具有相同的优先级,那么结果通常是 `testvar == b`, 但是由于我们给了 `a_group` 更高的优先级,所以结果是 `testvar == a`.

Note

`ansible_group_priority` 只能在清单中设置,而不能在`group_vars/`中设置,因为该变量用于加载`group_vars`。

## 使用多个来源

通过命令行提供多个清单参数或配置 `ANSIBLE_INVENTORY`可以同时定位多个清单资源。当想要同时针对一个特定的操作以通常独立的环境为目标时，这非常有用，比如准备和生产环境。

像这样从命令行指定两个源:

```bash
ansible-playbook get_logs.yml -i staging -i production
```



### 汇总目录中的库存来源

还可以通过组合目录下的多个清单来源和来源类型来创建清单。这对于组合静态和动态主机并将它们作为一个清单进行管理很有用。以下清单结合了清单插件源，动态清单脚本和具有静态主机的文件：

```bash
inventory/
  openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  dynamic-inventory.py   # add additional hosts with dynamic inventory script
  static-inventory       # add static hosts and groups
  group_vars/
    all.yml              # assign variables to all hosts
```

可以像下面这样定位此清单目录：

```bash
ansible-playbook example.yml -i inventory
```

如果存在与其他库存来源之间的变量冲突或组依赖关系，则控制库存来源的合并顺序可能很有用。根据文件名按字母顺序合并清单，因此可以通过在文件前添加前缀来控制结果：

```bash
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-static-inventory       # add static hosts
  group_vars/
    all.yml                 # assign variables to all hosts
```



## 连接到主机:行为清单参数

如上所述，设置以下变量可控制Ansible与远程主机的交互方式。

主机连接：



> 当使用ssh连接插件时,Ansible不会公开允许用户和ssh进程之间进行通信的通道,以手动接受密码来解密ssh密钥,建议使用`ssh-agent`。

- ansible_connection

  主机的连接类型。这可以是任何ansible连接插件的名称。 SSH协议类型是 `smart`, `ssh` or `paramiko`。 默认为`smart`。

所有连接：

- ansible_host

  要连接的主机名，如果与您要为其赋予的别名不同。

- ansible_port

  连接端口号（如果不是默认值）（ssh为22）

- ansible_user

  连接到主机时要使用的用户名

- ansible_password

  用于验证主机的密码

特定于SSH连接：

- ansible_ssh_private_key_file

  ssh使用的私钥文件。如果使用多个密钥并且您不想使用SSH代理，则很有用。

- ansible_ssh_common_args

  此设置始终附加到**sftp**，**scp**和**ssh**的默认命令行中。`ProxyCommand`为特定主机（或组）配置时很有用。

- ansible_sftp_extra_args

  此设置始终附加在默认的**sftp**命令行中。

- ansible_scp_extra_args

  此设置始终附加在默认的**scp**命令行中。

- ansible_ssh_extra_args

  此设置始终附加在默认的**ssh**命令行中。

- ansible_ssh_pipelining

  确定是否使用SSH管道。

- ansible_ssh_executable (added in version 2.2)

  此设置将覆盖使用系统**ssh**的默认行为。

特权升级：

- ansible_become

  等同于`ansible_sudo`或`ansible_su`，允许强制特权升级

- ansible_become_method

  允许设置权限提升方法

- ansible_become_user

  等同于`ansible_sudo_user`或`ansible_su_user`，允许设置通过特权升级成为您的用户

- ansible_become_password

  等效于`ansible_sudo_password`或`ansible_su_password`，允许您设置特权升级密码

- ansible_become_exe

  等同于`ansible_sudo_exe`或`ansible_su_exe`，允许您为所选的升级方法设置可执行文件

- ansible_become_flags

  等同于`ansible_sudo_flags`或`ansible_su_flags`，允许您设置传递给所选升级方法的标志。也可以`ansible.cfg`在`sudo_flags`选项中全局设置

远程主机环境参数：

- ansible_shell_type

  目标系统`shell`类型

- ansible_python_interpreter

  目标系统`python`路径

- ansible \_*\_ interpreter

  适用于ruby或perl之类的东西，



Ansible-INI主机文件中的示例：

```ini
some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
```

### 非SSH连接



如上一节所属,Ansible通过SSH执行剧本,但它不限于这种连接类型.使用主机特定的参数`ansible conntection=`可以更改连接类型,可以使用以下非SSH的连接器:

**local**

该连接器可用于将剧本部署到控制机器本身。

**docker**

该连接器使用本地Docker客户端将剧本直接部署到Docker容器中。此连接器处理以下参数：

- ansible_host

  要连接的Docker容器的名称。

- ansible_user

  在容器内操作的用户名。用户必须存在于容器内。

- ansible_become

  If set to `true` the `become_user` will be used to operate within the container.

- ansible_docker_extra_args

  可能是一个字符串，其中包含Docker可以理解的，不是特定于命令的任何其他参数。此参数主要用于配置要使用的远程Docker守护程序。

这是如何立即部署到创建的容器的示例：

```
- name: create jenkins container
  docker_container:
    docker_host: myserver.net:4243
    name: my_jenkins
    image: jenkins

- name: add container to inventory
  add_host:
    name: my_jenkins
    ansible_connection: docker
    ansible_docker_extra_args: "--tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/client-cert.pem --tlskey=/path/to/client-key.pem -H=tcp://myserver.net:4243"
    ansible_user: jenkins
  changed_when: false

- name: create directory for ssh keys
  delegate_to: my_jenkins
  file:
    path: "/var/jenkins_home/.ssh/jupiter"
    state: directory
```





## 清单设置示例



如果您需要管理多个环境，有时明智的做法是每个清单仅定义一个环境的主机。这样，例如，当您实际要更新某些“登台”服务器时，很难意外地更改“测试”环境中节点的状态。

对于上面提到的示例，您可以有一个 `inventory_test`文件：

```ini
[dbservers]
db01.test.example.com
db02.test.example.com	

[appservers]
app01.test.example.com
app02.test.example.com
app03.test.example.com
```

该文件只包含作为测试环境一部分的主机。在另一个名为的文件中定义 `inventory_staging`:

```ini
[dbservers]
db01.staging.example.com
db02.staging.example.com

[appservers]
app01.staging.example.com
app02.staging.example.com
app03.staging.example.com
```

要将一个名为Playbook的脚本应用于`site.yml` 测试环境中的所有应用服务器，请使用以下命令：

```bash
ansible-playbook -i inventory_test site.yml -l appservers
```



###　按功能分组

在上一节中，您已经看到了使用组对具有相同功能的主机进行集群的示例。例如，这使您可以在剧本或角色中定义防火墙规则，而不会影响数据库服务器：

```yaml
- hosts: dbservers
  tasks:
  - name: allow access from 10.0.0.1
    iptables:
      chain: INPUT
      jump: ACCEPT
      source: 10.0.0.1
```



### 按位置分组



```ini
[dc1]
db01.test.example.com
app01.test.example.com

[dc2]
db02.test.example.com
```

