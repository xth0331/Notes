# Jenkins Pipeline

## Jenkins Pipeline



Jenkins Pipeline是一套插件，支持在Jenkins中实现和集成*持续交付管道*。

一个持续交付（CD）管道是将软件从版本控制直接传递给用户和客户的过程的自动化表达。对软件的每次更改（在源代码管理中提交）都会在发布的过程中经历一个复杂的过程。此过程涉及以可靠且可重复的方式构建软件，以及通过多个测试和部署阶段推进构建的软件。

Pipline提供了一组可拓展的工具，用于通过管道特定域语言（DSL）语法“作为代码”对简单到复杂的传输管道进行建模。

Jenkins Pipeline 的定义被写入一个文本文件（Jenkinsfile），该文件可以提交给项目的源代码控制库。这是“Pipeline as code”的基础；将CD管道视为应用程序的一部分，以便像任何其他代码一样进行版本控制和审查。

创建`jenkinsfile`并将其提交给源代码管理提供了许多直接的好处：

- 自动为所有分支和拉取和请求创建Pipeline构建过程。
- Pipeline上的代码审查/迭代（以及剩余的源代码）。
- Pipeline的审计跟踪。
- Pipeline的单一数据来源，可由项目的多个成员查看和编辑。

虽然在Web UI或`Jenkinsfile`中定义Pipeline的语法是相同的，但通常认为最佳做法是在`Jenkinsfile`中定义Pipeline并检查其中的源码管理。

## 声明式与校本化Pipeline语法

一个`Jenkinsfile`可以使用两种语法编写 - Declarative和Scripted。

生命性和脚本化Pipeline的构造从根本上不同，生命性管道是Jenkins管道的最新特性，它具有：

- 提供比Scripted Pipeline语法更丰富的语法功能，以及
- 旨在使写作和阅读Pipeline代码更容易。



然而，写入`Jenkinsfile`文件的许多单独的语法组件（步骤）对应Declarative和ScriptedPipeline都是通用的。在下面的管道概念和管道语法概述中，详细了解这两种语法的不同之处。

## Why Pipeline？

从根本上说，Jenkins是一个支持多种自动化模式的自动化引擎。Pipeline为Jenkins添加了一套功能强大的自动化工具，支持从简单的持续集成到全面的CD Pipeline的用例。通过对一系列相关任务建模，用户可以利用Pipeline的许多功能：

- 代码：管理在代码中实现，通常检查到源代码控制，使团队能够编辑，审查和迭代其交付Pipeline。
- 持久： Pipeline可以在Jenkins master 的计划内核计划外重启中存活。
- 暂停：在继续Pipeline运行之前，Pipeline可以选择停止等待输入或批准。
- 多功能：Pipeline支持复杂的实际CD要求，包括并行分支/连接，循环和执行工作的能力。
- 可拓展：Pipeline插件支持其DSL自定义拓展和多个与其他插件集成的选项。

虽然Jenkins总是允许将自由式工作链接在一起的基本形式来执行顺序任务，Pipeline使这个概念成为Jenkins的一等公民。

基于Jenkins和兴的可拓展性，Pipeline也可以通过*Pipeline shared libraries*和插件开发人员进行拓展。

下面的刘承宇是在Jenkins管道中轻松建模的一个CD场景的示例：

![CD ](https://blog-image.nos-eastchina1.126.net/realworld-pipeline-flow.png)

## Pipeline概念

> 以下概念是Jenkins Pipeline的关键方面，它与Pipeline语法密切相关

### Pipeline

Pipeline是CD Pipeline的用户定义模型。Pipeline的代码定义了整个构建过程，通常包括构建应用程序，测试然后交付应用程序的阶段。

*此外，`pipeline`block 是Declarative Pipeline语法的关键部分。*

### Node

Node是一个机器，是Jenkins环境的一部分，能够执行Pipeline。

*此外，`node` block是Scripted Pipeline语法的关键部分。*

