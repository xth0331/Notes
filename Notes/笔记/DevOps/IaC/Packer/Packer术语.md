# Packer术语

- `Artifacts`（工件）

  一次构建的结果，通常是代表主机镜像的一组ID或文件。每个构建器都会生成一个`artifact`。例如，在EC2构建器中，`artifact`是一组AMI ID。对于VMware构建器，`artifact`是一个文件目录，其中包含创建的虚拟机。

- `Builds`（构建）

  是单个任务，最终为单个平台生成映像。多个构建并行运行。`Packer`构建产生一个AMI来运行我们的web程序，或`Packer`现在正在运行针对VMware，AWS和VirtualBox的构建。

- `Builders`（构建器）

  是`Packer`的组件，它们能够为单个平台创建映像。`Builder`读取了一些配置，并使用它们来运行和生成映像。为了创建实际的映像。`Builder`作为`Build`的一部分被调用，一遍创建实际的结果映像。

- `Commands`

  是执行某些工作的打包程序的子命令。一个示例命令是`Build`它作为打包程序的`Build`被调用。`Packer`开箱即用附带了一组命令，以定义其命令行界面。

- `Post-processors`

  `Packer`的组件，它们使用`Builder`或其他后处理的结果进行处理，以创建新的`artifact`。`post-processor`的示例包括压缩，上载等。

- `Provisioners`

  用于在运行的计算机上将软件安装和配置为静态映像之前对其进行安装和配置，它们执行使图像包含软件，包括`Shell`、`Chef`、`Puppet`等。

- `Termlates`

  是`Json`文件，它们通过配置`Packer`的各种组件来定义一个或多个`Build`。`Packer`能够读取模板并使用该信息来并行创建多个机器映像。



