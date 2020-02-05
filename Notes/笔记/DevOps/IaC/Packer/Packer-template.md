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

除了类型之外，其他键配置构建器本身。例如，AWS builder需要访问密钥、密钥和其他一些设置。这些直接放置在构建器定义中。

```json
{
  "type": "amazon-ebs",
  "access_key": "...",
  "secret_key": "..."
}
```

### 命令构建

Packer中的每个build都有一个名称。默认情况下，名称只是所使用的构建器的名称。总的来说，这已经足够了。名称仅充当正在发生的事情的一个指示器。但是，如果需要，可以在构建器定义中是有个`name`键指定定制名称。

如果您自定义了多个使用同一基础构建器的构建，则此功能特别有用。在这种情况下，必须为其中至少一个指定名称，因为名称必须唯一。

## 通信

每个构建都与单个通信器关联。通信器用于建立用于预配远程计算机的连接。

各种构建器的所有示例都显示了一些通信器（通常是SSH），但是通信器是高度可定制的。

## 模板通信器

通信器是Packer用于与正在创建的计算机上载文件，执行脚本等的机制。

在构建器部分配置了通信器。

所有通信器都有以下选项：

- `communicator`(string) - 目前Packer支持三种通信器：

  - `none` - 不会使用任何通信器。如果设置了，则大多数供应器也不能使用。
  - `ssh` - 将与计算机建立SSH连接。这通常是默认设置。
  - `winrm` - 将建立WinRM连接。

  除了上述内容以外，某些构建器还具有可以使用的自定义通信器。例如，`docker`构建器有一个`docker`通信器，该通信器使用`docker exec `和`docker cp`执行脚本和文件复制。
  
- `pause_before_connecting`（持续时间字符串|例如：“1h5m2s”）- 我们建议在客户端的引导脚本最后一步启用SSH或WinRM，但时有时可能遇到竞争，需要Packer等待尝试连接到guest。

  如果最终要到这种情况，可以使用模板`template`的`pause_before_connecting`。默认情况下，没有暂停，例如：

  ```json
  {
      "communicator": "ssh",
      "ssh_username": "myuser",
      "pause_before_connecting": "10m"
    }
  ```

  在此示例中，Packer将正常检测其是否可以连接。但是，一旦连接尝试成功，它将断开连接，然后等待十分钟，然后再连接到guest并开始配置。

### 准备使用通信器

根据构建器的不同，通信器可能无法具备 **开箱即用**工作所需的全部功能。

如果是从云映像构建的（例如，在Amazon上构建），那么云提供商很有可能有经在映像上预先配置了SSH，这意味着要做的就是在Packer模板配置通信器。



## 模板引擎

模板中的所有字符串均由Packer模板引擎处理，变量和函数可在运行时用于修改配置参数的值。

模板的语法使用以下约定：

- 与模板相关的所有事情都放在双括号内：`{{}}`。
- 函数直接在花括号内指定，例如`{{timestamp}}`。
- 模板变量以句号为前缀并大写，例如`{{.Variable}}`

### 功能

函数在字符串上和字符串内执行操作，例如，`{{timestamp}}`函数可在任何字符串中使用以生成当前时间戳。这对于需要唯一密钥的配置（例如AMI）很有用。通过将AMI名称设置为`My Packer AMI {{tmiestamp}}`，可以将AMI名称唯一化。如果需要大于秒的粒度，则应使用`{{uuid}}`，例如，当统一模板中有多个构建器时。

这是可供参考的功能的完整列表：

- `build_name` - 正在运行的构建的名称。
- `build_type` - 当前使用的构建器的类型。
- `clean_resource_name` - 名称只能包含某些字符且具有最大长度，这取决于平台。`clean_resource_name`会将大写字母转换为小写字母，并将非法字符替换为`-`例如：`"mybuild-{{isotime | clean_resource_name}}"`将成为 `mybuild-2017-10-18t02-06-30z`。
- `env` - 返回环境变量。
- `build` - 此引擎将允许您访问提供链接信息和基本示例状态信息的特殊标量。
-  `isotime` - UTC时间，可以将其格式化。
-  `lower` - 小写字符串。
-  `packer_version` - 返回Packer版本。
-  `pwd` - 执行Packer时的工作目录。
-  `replace` - （old, new string, n int, s）Replice返回字符串s的副本，其中旧的前n个非重叠实例都被new替换。
-  `replace_all` - (old, new string, s)  RepliceAll返回字符串s的副本，其中旧的所有非重叠实例都被new替换。
-  `split` - 使用分隔符分隔分割输入字符并返回请求的子字符串。
-  `template_dir` - 构建模板的目录。
-  `timestamp` - 当前UTC Unix时间戳。
-  `uuid` - 返回随机的UUID。
-  `upper` - 将字符串大写。
-  `user` - 指定用户变量。

### 模板变量

模板变量是Packer在构建时自动设置的特殊变量。一些构建器，预配器和其他组件具有仅可用于该组件的模板变量。模板变量是可识别的，因为它们以`.`为前缀，例如`{{ .Name }}`，在使用shell构建器模板时，可使用变量来自定义`execute_command`用于确定Packer将如何运行shell命令的参数。

```json
{
    "provisioners": [
        {
            "type": "shell",
            "execute_command": "{{.Vars}} sudo -E -S bash '{{.Path}}'",
            "scripts": [
                "scripts/bootstrap.sh"
            ]
        }
    ]
}
```

在`{{ .Vars }}`和`{{ .Path }}`模板变量将环境变量列表和路径被分别执行的脚本来代替。

## 模板Post-processors

模板中的后处理部分配置将对构建器构建的图像执行的任何后处理。后处理的示例将是压缩文件，上传工件等。

后处理器是可选的。如果模板中未定义任何后处理器，则将不对图像进行任何后处理。生成的结果工件只是生成器输出的图像。

在模板中，后处理器定义的一部分如下所示：

```json
{
  "post-processors": [
    // ... one or more post-processor definitions here
  ]
}
```

对于每个后处理器定义，Packer将获取每个已定义构建器的结果，并将其通过后处理器发送。这意味着，如果在模板中定义了一个后处理器，并且定义了两个构建器，则默认情况下，后处理器将运行两次（每个构建器一次）。如果愿意，有一些方法将在以后介绍，以控制构建器后处理器适用于什么。也可以防止后处理器运行。

### 后处理器定义

在`post-processors`模板的数组内，有三种方法定义后处理器。有简单的定义，详细的定义和序列定义。对比进行考虑的另一种方式，“简单”和“详细”定义是“序列”定义的快捷方式。

一个**简单的定义**就是一个字符串。后处理器的名称。一个例子如下所示。当后处理器不需要其他配置时，将使用简单定义。

```json
{
    "post-processors": ["compress"]
}
```

详细定义是JSON对象。它与构建者或供应者的定义非常相似。它包含一个`type`表示后处理器类型的字段，但也可能包含后处理器的其他配置。当需要除后处理器类型以外的其他配置时，使用详细定义。一个例子如下所示。

```json
{
  "post-processors": [
    {
      "type": "compress",
      "format": "tar.gz"
    }
  ]
}
```

序列定义有其他简单或详细定义组成的JSON数组。数组中定义的后处理器按顺序运行，每个后处理器的工件被输入下一个后处理器，任何中间工件都被丢弃。一个序列定义可能不包含另一个序列定义。序列定义用于将多个后处理器连接在一起。下面显示了一个示例，其中对构建的工件进行了压缩，然后上传，但是没有保留压缩后的结果。

```json
{
  "post-processors": [
    [
      "compress",
      { "type": "upload", "endpoint": "http://example.com" }
    ]
  ]
}
```

您可能会想到，简单的和详细的只是一个元素的序列定义的一种快捷方式。

### 输入工件

当使用后处理器是，输入工件（来自一个构建器或另一个后处理器）在后处理器运行后被默认丢弃。这是因为通常默认情况下，不希望中间工件在成为最终工件的过程中被创建。

但是，某些情况下，可能需要保留中间工件。可以通过将`keep_input_artifact`设置为`true`来告诉`Packer`保留这些工件。下面是一个示例。

```json
{
  "post-processors": [
    {
      "type": "compress",
      "keep_input_artifact": true
    }
  ]
}
```

此设置将仅将输入工件保留到该特定的后处理器。如果指定后处理器的序列，则默认情况下将丢弃所有中间，但后处理器的输入工件除外，后者明确声明要保留工件。

### 在特定版本上运行

可以使用`only`或`except`字段仅针对特定内部版本运行后处理器。这两个字段符合期望：仅将在指定的内部版本上运行后处理器，而除了将在指定的内部版本上运行后处理器。一系列后处理器，直到跳过最后一个后处理器位置。

下面显示了`only`被使用的示例,但`except`的用法实际上是相同的。`only`和`except`只能在详细字段中指定。如果要运行一个后处理器序列，则`only`和`except`将影响该后处理器并停止该序列。

`-except`选项可以专门跳过命名的后处理器。`-only`选项将忽略后处理器。

```json
[
  {
    // can be skipped when running `packer build -except vbox`
    "name": "vbox",
    "type": "vagrant",
    "only": ["virtualbox-iso"]
  },
  {
    "type": "compress" // will only be executed when vbox is
  }
],
[
  "compress", // will run even if vbox is skipped, from the same start as vbox.
  {
    "type": "upload",
    "endpoint": "http://example.com"
  }
  // this list of post processors will execute
  // using build, not another post-processor.
]
```

`only`或`except`中的值是构建名称，而不是生构建器类型。默认情况下，构建名称只是器构建器类型，但是，如果指定自定义名称参数，则应使用该名称作为值，而不是类型。

## 模板供应商

