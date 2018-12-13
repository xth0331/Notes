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

