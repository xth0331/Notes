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

## `build`

`packer build`命令获取一个模板并运行其中的所有构建，以生成一组工件，除非另有说明，否则在模板内指定的各种内部版本将并行执行。创建的工件在构建结束时输出。

### 选项

- [`-color=false`](https://www.packer.io/docs/commands/build.html#color-false) - 禁用彩色的输出，默认启用。
- [`-debug`](https://www.packer.io/docs/commands/build.html#debug) - 禁用并行化并启用调试模式。 调试模式标记构建器它们应该输出调试信息。 调试模式的确切行为留给了构建器。 通常，构建器通常会在每个步骤之间停止，等待键盘输入再继续。 这将允许用户检查状态等。
- [`-except=foo,bar,baz`](https://www.packer.io/docs/commands/build.html#except-foo-bar-baz) - 运行所有的构建和后处理程序，除了那些使用逗号分隔的名称的构建和后处理程序。默认情况下，构建和后处理器名称是它们的类型，除非在配置中指定了特定的name属性。跳过后处理器之后的任何后处理器都不会运行。因为后处理器可以嵌套在数组中，所以仍然可以运行不同的后处理器链。具有空名称的后处理器将被忽略。
- [`-force`](https://www.packer.io/docs/commands/build.html#force) - 当来自先前构建的构件阻止构建运行时，强制构建器运行。强制构建的确切行为留给构建者。通常，支持强制构建的构建器将从以前的构建中删除构件。这将允许用户重复构建，而不必预先手动清理这些工件。
- [`-on-error=cleanup`](https://www.packer.io/docs/commands/build.html#on-error-cleanup) (default), `-on-error=abort`, `-on-error=ask` - 选择构建失败时要做什么. `cleanup` 在完成上述步骤后进行清理，删除临时文件和虚拟机. `abort` 没有进行任何清理就退出，这可能需要下一次构建才能使用 `-force`. `ask` 出现提示并等待您决定清理，中止或重试失败的步骤.
- [`-only=foo,bar,baz`](https://www.packer.io/docs/commands/build.html#only-foo-bar-baz) - 仅使用给定的逗号分隔的名称运行构建. 默认情况下，构建名称是它们的类型，除非在配置中指定了特定的`name`属性。 `-only` 不适用于 post-processors.
- [`-parallel-builds=N`](https://www.packer.io/docs/commands/build.html#parallel-builds-n) - 限制要并行运行的内部版本数，0表示没有限制（默认为0）。
- [`-timestamp-ui`](https://www.packer.io/docs/commands/build.html#timestamp-ui) - 使用RFC3339时间戳为每个ui输出启用前缀。
- [`-var`](https://www.packer.io/docs/commands/build.html#var) - 在Packer程序模板中设置一个变量。 此选项可以多次使用。 这对于设置内部版本号很有用。
- [`-var-file`](https://www.packer.io/docs/commands/build.html#var-file) - 从文件中设置模板变量。



## `console`选项

`packer console`命令允许尝试使用Packer变量。可以在调用控制台的Packer配置中访问变量，也可以在使用`-var` 或`-var-file`命令行选项中调用控制台时提供变量。

键入插值进行测试，然后按<kbd>Enter</kbd> 或使用<kbd>Control</kbd> + <kbd>C</kbd>

```bash
packer console my_template.json
```

控制台命令将接受的选项的完整列表在帮助输出中可见，可以通过`packer console -h`看到。

### 选项 

- `-var` 在冲中模板中设置一个变量。此选项可以多次使用。这对于设置内部版本很有用。例如：`-var “myvar=asf”`
- `-var-file`  从文件中设置模板变量。例如： `-var-file myvars.json`

### 使用示例

假设使用Packer模板`example_template.json`启动控制台:

```bash
packer console example_template.json
```

将进入提示，允许输入模板函数并查看它们。例如，如果变量在`example_template`的变量部分中定义：

```json
"variables":{
    "myvar": "asdfasdf"
},
...
```

然后{{user `myvar`}}在Packer console中输入，将看到`myvar`的值：

```json
> {{user `myvar`}}
> asdfasdf
```

从那里您可以测试更复杂的插值：

```
> {{user `myvar`}}-{{timestamp}}
> asdfasdf-1559854396
```

使用完控制台后，只需键入“exit”或CTRL-C

```bash
> exit
$
```

如果要提供变量或变量文件，请执行以下操作：

```bash
packer console -var "myvar=fdsafdsa" -var-file myvars.json example_template.json
```

如果您没有要测试的特定变量或var文件，而只想尝试使用特定的模板引擎，则可以通过简单地调用`packer console`而无需模板文件来进行操作。

如果您只想查看特定的单个插值而不启动REPL，则可以通过将字符串回显并将其传递到控制台命令中来实现：

```bash
echo {{timestamp}} | packer console
1559855090
```

## `fix`命令

`packer fix`命令采用一个模板，找到向后不兼容的部分， 并使其更新，所以它可以适用于最新版本的Packer。更新到新版的Packer后，应该运行fix命令还确保模板与新版本是否兼容。

`fix`命令将已更改的模板输出为标准输出，因此如果想要保存到文件中，则应使用特定于标准操作系统的重定向。例如，LInux上，需要执行以下操作：

```bash
packer fix old.json > new.json
```

如果由于任何原因修复失败，则fix命令将以非0退出状态退出。错误消息出现在标准错误上，因此如果重定向输出，仍会看到错误消息。

### 选项

- `-validate=false` =禁用对固定模板的验证。默认为`True`。

## `inspect`命令

`packer inspect`命令接受一个模板并输出定义的各种组件。这可以帮助快速了解模板，而不必深入了解JSON本身。该命令将告诉告诉您模板接受哪些变量，定义哪些构建器、它定义的`provisioner`以及它们运行的顺序等。

当启用机器可读输出时，此命令特别有用。该命令以机器可解析的方式输出。

不会验证各个组件的实际配置，但是会根据需要验证模板语法。

### 例子

给定一个基本模板，下面是一个输出示例：

```bash
packer inspect template.json
Variables and their defaults:

  aws_access_key =
  aws_secret_key =

Builders:

  amazon-ebs
  amazon-instance
  virtualbox-iso

Provisioners:

  shell
```

## `validate`命令

`packer validate`命令用于验证模板的语法和配置。该命令将在成功时返回0的退出状态码，在失败时返回非0。此外，如果模板没有验证，则将输入错误消息。

例如：

```bash
packer validate my-template.json
Template validation failed. Errors are shown below.

Errors validating build 'vmware'. 1 error(s) occurred:

* Either a path or inline script must be specified.
```

### 选项

- `-syntax-only` 仅检查模板的语法。
- `-except=foo,bar,vaz` 生成所有带有给定逗号分隔名称的构建和`post-processors`，除了那些用逗号分隔的名称是它们的构建器名称，除非在配置中指定了特定的`name`属性。具有空名称的`post-processor`将被忽略。
- `-only=foo,bar,baz`  仅使用给定的逗号分隔的名称构建构建 默认情况下,构建名称是它们的构建器的名称,除非在配置中指定了特定的name属性。
- `-var` 在packer程序模板中设置一个变量。此选项可以使用多次，这对于设置内部版本很有用。
- `-var-file`  从文件中设置模板变量。

