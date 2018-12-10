# 创建一个RPM包

创建RPM包可能很复杂。下面是一个简化的示例:

```specfile
Name:       hello-world
Version:    1
Release:    1
Summary:    Most simple RPM package
License:    FIXME

%description
This is my first RPM package, which does nothing.

%prep
# we have no source, so nothing here

%build
cat > hello-world.sh <<EOF
#!/usr/bin/bash
echo Hello world
EOF

%install
mkdir -p %{buildroot}/usr/bin/
install -m 755 hello-world.sh %{buildroot}/usr/bin/hello-world.sh

%files
/usr/bin/hello-world.sh

%changelog
# let skip this for now
```

将文件另存为`hello-world.spec`。

安装依赖：

```bash
yum -y install gcc rpm-build rpm-devel rpmlint make python bash coreutils diffutils patch rpmdevtools
```

然后执行：

```bash
rpmdev-setuptree
rpmbuild -ba hello-world.spec
```

`rpm-setuptree`命令创建了几个工作目录。由于这些目录永久存储在`$HOME`中，因此不需要再次输入命令。

执行`rpm-build`命令的输出类似：

```bash
执行(%prep): /bin/sh -e /var/tmp/rpm-tmp.mwA7kE
+ umask 022
+ cd /root/rpmbuild/BUILD
+ exit 0
执行(%build): /bin/sh -e /var/tmp/rpm-tmp.WRcFAO
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cat
+ exit 0
执行(%install): /bin/sh -e /var/tmp/rpm-tmp.UQtmSY
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64
++ dirname /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64/usr/bin/
+ install -m 755 hello-world.sh /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64/usr/bin/hello-world.sh
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
处理文件：hello-world-1-1.x86_64
Provides: hello-world = 1-1 hello-world(x86-64) = 1-1
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Requires: /usr/bin/bash
检查未打包文件：/usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64
写道:/root/rpmbuild/SRPMS/hello-world-1-1.src.rpm
写道:/root/rpmbuild/RPMS/x86_64/hello-world-1-1.x86_64.rpm
执行(%clean): /bin/sh -e /var/tmp/rpm-tmp.1fjrRw
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/hello-world-1-1.x86_64
+ exit 0
```

`$HOME/rpmbuild/RPMS/x86-64/hello-world-1-1.x86_64.rpm`是生成的RPM包。