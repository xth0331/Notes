# Linux安装JAVA JDK

**本教程可帮助您在系统上安装Java 8或更新Java。在从Linux命令行下载Java之前，请仔细阅读说明。**

|             |                                                              |
| ----------- | ------------------------------------------------------------ |
| JDK下载链接 | https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html |





## Step 1 – 下载最新的Java

Oracle团队提供Java RPM包以及编译的源代码。很多次我尝试使用rpm包安装Java，但是我遇到了一些问题。所以我决定使用已编译的源代码安装Java。从那以后，我在CentOS上安装了大量Java，基于Redhat的系统没有任何问题。要从其[官方下载页面](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载最新的Java SE Development Kit 8版本，或使用以下命令从shell下载。

```bash
cd /opt/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz"
tar xzf jdk-8u201-linux-x64.tar.gz
```

## Step 2 – 使用Alternatives安装Java 8

alternatives命令用于维护符号链接。此命令用于创建，删除，维护和显示有关包含备选系统的符号链接的信息。让我们使用`alternatives`命令在您的系统上配置Java。`alternative`命令在**chkconfig**包中可用。

```bash
cd jdk1.8.0_201/
alternatives --install /usr/bin/java java /opt/jdk1.8.0_201/bin/java 2
alternatives --config java
```

新安装的Java版本列在第n位，因此输入n并按Enter键。

```
There are 3 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
   1           /opt/jdk1.8.0_45/bin/java
 * 2           /opt/jdk1.8.0_144/bin/java
 + 3           /opt/jdk-11/bin/java
   4           /opt/jdk1.8.0_201/bin/java

Enter to keep the current selection[+], or type selection number: 4
```

此时，JAVA 8已成功安装在您的系统上。我们还建议使用替代方法设置javac和jar命令路径。

```
alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_201/bin/jar 2
alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_201/bin/javac 2
alternatives --set jar /opt/jdk1.8.0_201/bin/jar
alternatives --set javac /opt/jdk1.8.0_201/bin/javac
```

## Step 3 – 检查已安装的Java版本

Java和javac二进制文件在PATH环境变量下可用。您可以在系统中的任何位置使用它们。让我们通过执行以下命令检查系统上安装的Java运行时环境（JRE）版本。

```bash
java -version

java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```

## Step 4 – 设置Java环境变量

大多数基于Java的应用程序使用环境变量来工作。使用以下命令设置Java环境变量

设置**JAVA_HOME**，**JRE_HOME**和**PATH**环境变量。

```bash
export JAVA_HOME=/opt/jdk1.8.0_201
export JRE_HOME=/opt/jdk1.8.0_201/jre
export PATH=$PATH:/opt/jdk1.8.0_201/bin:/opt/jdk1.8.0_201/jre/bin
```

还要将上述命令添加到`/etc/bashrc`或`/etc/profile`文件中以自动设置环境变量。