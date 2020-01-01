# Packer CLI

 `Packer`使用命令行界面控制。与Packer的所有互动都通过`packer`工具完成。像许多其他命令行工具一样，`packer`工具需要一个子命令来执行，并且该子命令可能还有其他选项。子命令由`packer SUBCOMMAND`执行，其中`SUBCOMMAND`是希望执行的实际命令。

如果自己运行`packer`，将显示所有可用子命令和他们的简短概要。除此之外，还可以运行带有`-h`的任何`packer`命令，以输出特定子命令更详细的帮助。

##  机器可读的输出

默认情况下，Packer的输出非常易于阅读。它使用漂亮的格式，间距和颜色，以使Packer可以轻松使用。但是，Packer的构建考虑了自动化。为此，Packer支持完全机器可读的输出设置，可以在自动化环境轻松使用Packer。

因为机器可读的输出格式是考虑到Unix工具的，例如`awk/sed/grep`等等。友好且提供熟悉的界面，无需学习新格式。

### 启用机器可读的输出

机器可读的输出格式可以i通过传递`-machine-readable`标志给任何Packer命令来启用。这立刻使得所有输出在`stdout`上成为机器可读的。如果启用了日志记录，它将继续出现在`stderr`上。输出示例如下：

```bash
packer -machine-readable version
1498365963,,version,1.0.2
1498365963,,version-prelease,
1498365963,,version-commit,3ead2750b+CHANGES
1498365963,,ui,say,Packer v1.0.2
```

### 机器可读输出的格式化

机器可读格式是面向行的逗号分隔的文本格式。除了像Ruby或Python这样完整的编程语言之外，这还使使用`awk`或`grep`等标准Unix工具进行解析更加方便。

格式为：

```bash
timestamp,target,type,data...
```

每个组件说明如下：

- `timestamp` 实在UTC中打印消息时的Unix时间戳。
- `target`当你调用`packer build`时，它可以式空的，也可以是单独的`build`名，例如`amazon-ebs`。当`build`正在进行时，它通常是空的，当特定`build`的构件被引用时，它是构件名称。
- `type` 是输出的机器可读消息的类型。最常见的两种类型是`ui`和`artifact`。
- `date` 是与前一个类型关联的另个或多个逗号分隔值。此数据的确切数量和含义取决于类型。

在格式内，如果数据包含逗号，则将其替换为`%(PACKER_COMMA)` 。这比`\`之类的转移字符更受欢迎，因此它对`awk`之类的工具更友好。

格式中的换行符将替换为各自的标准转义顺序。换行符在输出中变成一个`\n`。回车符变成`\r`。

### 机器可读的消息类型

下面是您可能在机器可读输出中可能看到的类型的不完整列表：

运行`packer build`时，将看到这些数据类型：

- `ui`：这意味着所提供的信息是人类可读的字符串，即使我们不在机器可读的模式下，该字符也将被发送到`stdout`。有三种与该类型关联的`data`子类型：
  - `say`: 在非机器可读格式中，这将被加粗。通常，它用于发布有关构建过程中开始新步骤的声明。
  - `message`: 最常用的消息类型，用于在构建过程中进行基本更新。
  - `error`: 为错误保留
- `artifact-count`: 这个数据类型告诉您一个特定的构建产生了多少工件。
- `artifact`: 此数据类型告诉您有关`Packer`在其构建期间创建的内容的信息。输出的示例遵循模式时间戳、构建名、工件、工件编号、键、值，其中键和值包含有关工件的信息。

```bash
  1539967803,,ui,say,\n==> Builds finished. The artifacts of successful builds are:
  1539967803,amazon-ebs,artifact-count,2
  1539967803,amazon-ebs,artifact,0,builder-id,mitchellh.amazonebs
  1539967803,amazon-ebs,artifact,0,id,eu-west-1:ami-04d23aca8bdd36e30
  1539967803,amazon-ebs,artifact,0,string,AMIs were created:\neu-west-1: ami-04d23aca8bdd36e30\n
  1539967803,amazon-ebs,artifact,0,files-count,0
  1539967803,amazon-ebs,artifact,0,end
  1539967803,,ui,say,--> amazon-ebs: AMIs were created:\neu-west-1: ami-04d23aca8bdd36e30\n
  1539967803,amazon-ebs,artifact,1,builder-id,
  1539967803,amazon-ebs,artifact,1,id,
  1539967803,amazon-ebs,artifact,1,string,
  1539967803,amazon-ebs,artifact,1,files-count,0
  2018/10/19 09:50:03 waiting for all plugin processes to complete...
  1539967803,amazon-ebs,artifact,1,end
```

运行`packer version`时，您将看到这类数据类型：

- `version`: Packer是什么版本
- `version-prerelease`: 如果版本是预发行版，则数据包含`dev`，否则为空
- `version-commit`: Packer分之当前正在进行的提交的git hash

## 自动补全

`packer`命令的特点是选择子命令自动完成。可以通过`packer -autocomplete-install`。这样做之后，可以调用一个新的并使用该特性。

例如，假设在每个提示行的末尾都键入一个选项卡：

```bash
packer p
plugin  build
packer build -
-color             -debug             -except            -force             -machine-readable  -on-error          -only              -parallel          -timestamp          -var               -var-file
```



