# Packer模板

模板是JSON文件，用于配置Packer的各个组件，以便创建一个或多个映像。模板是可移植的，静态的，并且人和计算机均可读写。这具有额外的好处，不仅可以手动创建和修改模板，还可以编写脚本来动态创建模板。

模板被提供给诸如`packer build`之类的命令，该命令将获取模板并实际在其中隐形构建，从而生成映像。

## 模板结构

模板是JSON对象，它具有一组配置Packer的各种组件的键。模板中可用的键如下。

- `builders`（*必须*）是一个由一个或多个对象组成的数组，该数组定义将用于为该模板创建机器映像的构建器，并配置每个构建器。
- `description`（*可选*）是一个字符串，它提供了模板的描述，此输出仅在检查命令中使用
- `min_packer_version` （*可选*）是一个字符串，该字符串具有解析模板所需的最低`packer`版本。由于Packer保留了与Packer fix的向后兼容性，因此无法指定最高版本。
- `post-processors`（*可选*）是由一个或多个对象组成的数组，这些对象定义了哟啊对生成的映像执行的各种后处理步骤。如果未指定，则不会进行任何后处理。
- `provisioners`(可选)是由一个或多个对象组成的数组，该数组定义将用于为每个构建器创建的计算机安装和配置软件的提供程序。如果没有指定它，则不会运行任何`provisioner`。
- `variables`(可选)是一个或多个key/value字符串的对象。这些key/value字符串定义了模板中包含的用户变量。如果未指定，则不会定义任何变量。

## 模板示例

下面是一个基本模板的示例，可以用packer构建程序调用它。它将在AWS中创建一个实例，并在运行后将脚本复制到实例并使用SSH运行该脚本。

> 此示例需要具有Amazon Web Service账号。要进行功能构建，需要提供许多参数。

```json
{
  "builders": [
    {
      "type": "amazon-ebs",
      "access_key": "...",
      "secret_key": "...",
      "region": "us-east-1",
      "source_ami": "ami-fce3c696",
      "instance_type": "t2.micro",
      "ssh_username": "ubuntu",
      "ami_name": "packer {{timestamp}}"
    }
  ],

  "provisioners": [
    {
      "type": "shell",
      "script": "setup_things.sh"
    }
  ]
}
```

## Template Builders

在模板中，`builder`部分包含一组数组，其中包含Packer用于生成模板的机器映像的所有构建器。

构建器负责为各种平台创建机器并从中生成映像。例如，EC2、VMware、VirtualBox等都有单独的构建器。默认情况下，Packer有许多生成器，也可以扩展以添加新的生成器。

在模板中，构建器定义的一部分如下所示：

```json
{
  "builders": [
    // ... one or more builder definitions here
  ]
}
```

### Builder定义

单个构建器定义映射到一个构建。构建器定义是至少需要一个类型的JSON对象。类型是将用于为构建创建机器映像的构建器的名称。

