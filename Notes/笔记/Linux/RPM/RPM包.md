# RPM包

## 什么是RPM？

RPM包只是一个包含系统所需的其他文件和信息的文件。具体来说，RPM包由包含文件的`cpio`存档和包含有关包的元数据的RPM标头组成。`rpm`软件包管理器使用这种元数据来确定的依赖。在那安装文件和其他信息。

有两种类型的RPM包：

- 源RPM（SRPM）
- 二进制RPM

SRPM和二进制RPM弓箭文件格式和工具，但具有不同的内容并用于不同的目的。SRPM包含源代码，可选的补丁和SPEC文件，它描述了如何将源代码构建为二进制文件。

### RPM打包工具

查看安装位置

```bash
yum -y install rpmdevtools
rpm -ql rpmdevtools
```

### RPM打包工作区

要设置作为RPM打包工作区的目录布局，请使用`rpmdev-setuptree`

```bash
rpmdev-setuptree
tree ~/rpmbuild/


rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS

5 directories, 0 files

```

创建的目录用于以下目的：

| 目录    | 目的                                                         |
| ------- | ------------------------------------------------------------ |
| BUILD   | 构建包时，会在此处创建`%buildroots`目录，如果日志输出未提供足够的信息，这对于查找失败原因有帮助。 |
| RPMS    | 二进制RPM在此处创建，位于不同体系结构的子目录中，例如在`x86_64`和`noarch`中 |
| SOURCES | 打包器放置压缩的源代码存档和补丁。`rpmbuild`在这里查找他们。 |
| SPECS   | 打包器将SPEC文件放在此处。                                   |
| SRPMS   | 当`rpmbuild`用于构建SRPM而不是二进制RPM时，将在此处创建生成SRPM。 |

### 什么是SPEC文件

SPEC文件可以被任务是`rpmbuild`程序用于构建RPM的方法。通过定义指令来告诉构建系统要做什么，这些部分在`Preamble`和`Body`中定义。`Preamble`包含`Body`中使用的一系列元数据项。`Body`包含说明的主要部分。

#### Preamble 项目

列出RPM SPEC文件中的Preamble部分的选项：

| SPEC指令        | 定义                                                         |
| --------------- | ------------------------------------------------------------ |
| `Name`          | 包的基本名称，应与SPEC文件名匹配                             |
| `Version`       | 软件的上游版本号                                             |
| `Release`       | 此版本软件发布的次数。通常，将初始值设为`1%{?dist}`，并在每个新版本的包中增加它。在构建新版本时重置为1。 |
| `Summary`       | 包的简短摘要。                                               |
| `license`       | 正在打包的软件的许可证                                       |
| `URL`           | 有关该程序的更多信息的完整URL，通常这是打包软件上游项目的网站。 |
| `Source0`       | 上游源代码的压缩归档的路径或URL。这应该指向存档的可访问且可靠的存储，例如上游页面而不是打包器的本地存储。如果需要，可以添加更多的SourceX,数字递增。例如：Source1。 |
| `Patch0`        | 必要时应用于源代码的第一个补丁的名称。如果需要，可以添加更多PatchX指令，每次递增数字，例如：Patch1，Patch2，Patch3等。 |
| `BuildArch`     | 如果包不依赖于体系结构，例如，如果完全使用解释型编程语言编写，请将其设置为`BuildArch: noarch`。如果未设置，则程序包会自动继承构建它的计算机的体系结构`x86_64`。 |
| `BuildRequires` | 以逗号或空格分隔的包列表，用于构建使用编译语言编写的程序。可以有多个`BuildRequires`条目。每个条目在SPEC文件中都有自己的行。 |
| `Requires`      | 软件包在安装后运行所需的软件包列表，可以有多条`Requires`条目，每个条目在SPEC文件中都有自己的行。 |
| `ExcludeArch`   | 如果某个软件无法在特定的处理器体系上运行，则可以在此处排除该体系结构。 |

`Name`,`Version`和`Release`指令包含RPM包的文件名。RPM包维护者和系统管理员经常将这三个指令称为`NVR`或`N-V-R`，因为RPM包文件名具有`NAME-VERSION-RELEASE`格式。

可以通过使用`rpm`查询特定的包来获取`NAME-VERSION-RELEASE`：

```bash
rpm -q python
python-2.7.5-76.el7.x86_64
```

这里`python`是Package Name，`2.7.5`是Version，`76.el7`是Release，`x86_64`是体系结构，与**NVR**不同，体系结构标记不受RPM包装程序的直接控制，而是由`rpmbuild`构建环境定义。例外是与架构无关的`noarch`包。