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

#### Preamble 项

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

####  Body 项

此表列出了RPM SPEC文件的Body部分中使用的项：

| SPEC指令       | 定义                                                         |
| -------------- | ------------------------------------------------------------ |
| `%descriptipn` | RPM中打包的软件的完整描述。该描述可以跨越多行并且可以分成段落。 |
| `%prep`        | 用于准备要构建的软件的命令或一系列命令，例如，解压缩存档`Source0`、该指令可以包含shell脚本。 |
| `%build`       | 用于将软件实际构建为机器码或字节码的命令或一系列命令。       |
| `%install`     | 用于将所需构建组件从`%builffit`（构建发生的位置）复制到`%buildroot`目录（包含要打包的文件的目录结构）的命令。通常意味着将文件从`~/rpmbuild/BUILD`复制到`~/rpmbuild/BUILDROOT`。这尽在创建包时运行，而不是在安装时运行。 |
| `%check`       | 用于测试软件的命令。通常包括单元测试等内容。                 |
| `%files`       | 将在最终用户的系统中安装的文件列表。                         |
| `changelog`    | 不同`Version`或`Release`之间发生的更改记录。                 |

### BuildRoots

在RPM打包的北京下，`buildroot`是一个chroot环境。这意味着使用与最终用户系统中相同的文件系统层次结构将构建组件放置在此处，其中`buildroot`充当根目录，构建组件的防止应符合最终用户系统文件层次结构标准。

`buildroot`中的文件稍后会被放入`cpio`存档中，后者将成为RPM的主要部分，在最终用户的系统上安装RPM时，会在根目录中提取这些文件，从而保留正确的层次结构。

### RPM宏

RPM宏是直接文本替换，可以在使用某些内置功能时根据语句的可选评估有条件的分配。这意味着您可以让RPM为您执行文本替换。

例如，在SPEC文件中多次引用打包软件版本时，这很有用。您只能在`%{version}`宏中定义一次版本。然后再整个SPEC文件中使用`%{version}`。每次出现都会自动替换之前定义的版本。

> 如果看到一个不熟悉的宏，可以使用一下方法对其进行评估：
>
> ```bash
> rpm --eval %{_MACRO}
> ```
>
> 例如：
>
> ```bash
> rpm --eval %{_bindir}
> rpm --eval %{_libexecdir} 
> ```
>
>

`%{?dist}`一个常见的宏，表示“分发标签”。表示用于构建的分布。

例如：

```bash
rpm --eval %{?dist}
```



### 使用SPEC文件

将软件打包成RPM的很多一部分工作时编辑SPEC文件。

要打包新软件，需要创建一个新的SPEC文件，不要从头开始手动编写，使用`rpmdev-new-spec`命令。它会创建一个未填充的SPEC文件，并填写必要的指令和字段。

```bash
rpmdev-newspec bello
```

生成的`bello.spec`内容如下：

```bash
Name:           bello
Version:
Release:        1%{?dist}
Summary:

License:
URL:
Source0:

BuildRequires:
Requires:

%description


%prep
%setup -q


%build
%configure
make %{?_smp_mflags}


%install
rm -rf $RPM_BUILD_ROOT
%make_install


%files
%doc



%changelog
```

完善`bello.spec`内容：

```bash
Name:           bello
Version:        0.1
Release:        1%{?dist}
Summary:        Hello World example implemented in bash script

License:        GPLv3+
URL:            https://www.example.com/%{name}
Source0:        https://www.example.com/%{name}/releases/%{name}-%{version}.tar.gz

Requires:       bash

BuildArch:      noarch

%description
The long-tail description for our Hello World Example implemented in
bash script.

%prep
%setup -q

%build

%install

mkdir -p %{buildroot}/%{_bindir}

install -m 0755 %{name} %{buildroot}/%{_bindir}/%{name}

%files
%license LICENSE
%{_bindir}/%{name}

%changelog
* Tue May 31 2016 Adam Miller <maxamillion@fedoraproject.org> - 0.1-1
- First bello package
- Example second item in the changelog for version-release 0.1-1
```

完善`pello.spec`内容：

```bash
Name:           pello
Version:        0.1.1
Release:        1%{?dist}
Summary:        Hello World example implemented in python

License:        GPLv3+
URL:            https://www.example.com/%{name}
Source0:        https://www.example.com/%{name}/releases/%{name}-%{version}.tar.gz

BuildRequires:  python
Requires:       python
Requires:       bash

BuildArch:      noarch

%description
The long-tail description for our Hello World Example implemented in
Python.

%prep
%setup -q

%build

python -m compileall %{name}.py

%install

mkdir -p %{buildroot}/%{_bindir}
mkdir -p %{buildroot}/usr/lib/%{name}

cat > %{buildroot}/%{_bindir}/%{name} <<-EOF
#!/bin/bash
/usr/bin/python /usr/lib/%{name}/%{name}.pyc
EOF

chmod 0755 %{buildroot}/%{_bindir}/%{name}

install -m 0644 %{name}.py* %{buildroot}/usr/lib/%{name}/

%files
%license LICENSE
%dir /usr/lib/%{name}/
%{_bindir}/%{name}
/usr/lib/%{name}/%{name}.py*

%changelog
* Tue May 31 2016 Adam Miller <maxamillion@fedoraproject.org> - 0.1.1-1
  - First pello package
```

完善`cello.spec`：

```bash
Name:           cello
Version:        1.0
Release:        1%{?dist}
Summary:        Hello World example implemented in C

License:        GPLv3+
URL:            https://www.example.com/%{name}
Source0:        https://www.example.com/%{name}/releases/%{name}-%{version}.tar.gz

Patch0:         cello-output-first-patch.patch

BuildRequires:  gcc
BuildRequires:  make

%description
The long-tail description for our Hello World Example implemented in
C.

%prep
%setup -q

%patch0

%build
make %{?_smp_mflags}

%install
%make_install

%files
%license LICENSE
%{_bindir}/%{name}

%changelog
* Tue May 31 2016 Adam Miller <maxamillion@fedoraproject.org> - 1.0-1
- First cello package
```

 `/etc/rpmdevtools/`目录中有几种流行语言SPEC文件模板。

## 构建RPMS

RPM是使用`rpmbuild`构建的。

方案：

1. 构建源RPM
2. 构建二进制RPM

`rpmbuild`需要某个目录和文件结构。这与`rpmdev-setuptree`程序设置的结构相同。

### 源RPMS

为什么要构建源RPM（SRPM）?

1. 保留不是道环境的RPM的某个Name-Version-Release的确切来源。这包括确切的SPEC文件，源代码和所有相关补丁。这对于回顾历史记录和调试很有用。
2. 能够在不同硬件平台或体系架构上构建二进制RPM。



#### 创建SRPM

```bash
rpmbuild -bs SPECFILE
```

这里，我们建立`bello`、`pello`、`cello`的SRPM。

```bash
cd ~/rpmbuild/SPECS/
rpmbuild -bs bello.spec
写道:/root/rpmbuild/SRPMS/bello-0.1-1.el7.centos.src.rpm
rpmbuild -bs cello.spec
写道:/root/rpmbuild/SRPMS/cello-1.0-1.el7.centos.src.rpm
rpmbuild -bs pello.spec
写道:/root/rpmbuild/SRPMS/pello-0.1.1-1.el7.centos.src.rpm
```



> 注意，SRPM被防止在`rpmbuild/SRPMS`目录中，



### 二进制RPMS

构建二进制RPM有两种方法：

1. 使用`rpmbuild --rebuild`命令重建。
2. 使用`rpmbuild -bb`命令从SPEC文件构建，`-bb`选项代表构建二进制。

#### 从源RPM重建

要从源RPM重`bello`、`cello`以及`pello`，运行：

```bash
rpmbuild --rebuild ~/rpmbuild/SRPMS/bello-0.1-1.el7.centos.src.rpm
rpmbuild --rebuild ~/rpmbuild/SRPMS/cello-1.0-1.el7.centos.src.rpm
rpmbuild --rebuild ~/rpmbuild/SRPMS/pello-0.1.1-1.el7.centos.src.rpm
```

- 创建二进制RPM时生成的输出是详细的，有助于调试
- 调用`rpmbuild --rebuild`：
  - 将SRPM内容安装到`~/rpmbuild/`。
  - 使用已安装的内容构建。
  - 删除SPEC和源码。

如果想要保留SPEC和源码，有以下两个办法：

1. 构建时使用`--recompile`而不是`--rebuild`。

2. 使用一下命令安装SRPM：

   ```bash
   rpm -Uvh 
   ```


#### 从SPEC构建二进制文件

```bash
rpmbuild --bb ~/rpmbuild/SPEC/bello.spec
rpmbuild --bb ~/rpmbuild/SPEC/cello.spec
rpmbuild --bb ~/rpmbuild/SPEC/pello.spec
```

## 检查RPM

使用`rpmlint`检查RPM包:

```bash
rpmlint ~/build/RPMS/x86_64/cello-1.0-1.el7.x86_64.rpm
```

使用`rpmlint`检查SPEC文件：

```bash
rpmlint ~/rpmbuild/SPECS/cello.spec
```

