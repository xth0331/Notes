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

### Stage

`stage` block定义了通过整个Pipeline执行的概念上不同的任务子集（例如“构建”、“测试”和“部署”阶段），许多插件使用它来可视化或呈现管道状态/进度。

### Step

一项任务。从根本上说，一个步骤告诉Jenkins在特定时间点做什么（或者在过程中“步骤”）。例如，要执行shell命令`make`用`sh`,setp: sh‘ make’。当插件拓展Pipeline DSL时，通常意味着插件已经实现了一个新步骤。

## Pipeline 语法概述

一下Pipeline代码框架说明了Declarative Pipeline语法和Scripted Pipeline语法之间的 区别。



### 声明式Pipeline基础

在Declarative Pipeline语法中，`pipeline`block定义了整个Pipeline中完成的所有工作。

Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any 
    stages {
        stage('Build') { 
            steps {
                // 
            }
        }
        stage('Test') { 
            steps {
                // 
            }
        }
        stage('Deploy') { 
            steps {
                // 
            }
        }
    }
}
```

| 参数         | 含义                                         |
| ------------ | -------------------------------------------- |
| `agent any ` | 在任何可用的代理上执行此Pipeline或其任何阶段 |
| `stage('Bulid')` | 定义构建阶段 |
| `//` | 执行阶段相关的一些步骤 |
| `stage('Test')` | 定义测试阶段 |
| `//` | 执行测试阶段相关的一些步骤 |
| `stage('Deploy)` | 定义部署阶段 |
| `//` | 执行部署阶段相关的 一些步骤 |

###　脚本式Pipeline基础

在Scripted Pipeline语法中，一个或多个`node`在整个Pipeline中执行核心工作，虽然这不是Scripted Pipeline语法的强制要求，但将Pipeline的工作限制在`node`块内部有两个作用：

1. 通过向Jenkins队列添加项目来计划要运行的块中包含的步骤。只要执行程序在节点上空闲，步骤就会运行。
2. 创建工作空间（特定于该特定Pipeline的目录），可以对从源控件检出的文件执行工作。

Jenkinsfile (Scripted Pipeline)

```json
node {  
    stage('Build') { 
        // 
    }
    stage('Test') { 
        // 
    }
    stage('Deploy') { 
        // 
    }
}
```



| 参数              | 含义                                         |
| ----------------- | -------------------------------------------- |
| `node`            | 在任何可用的代理上执行此Pipeline或其任何阶段 |
| `stage('Build')`  | 定义构建阶段                                 |
| `//`              | 执行阶段相关的一些步骤                       |
| `stage('Test')`   | 定义测试阶段                                 |
| `//`              | 执行测试阶段相关的一些步骤                   |
| `stage('Deploy')` | 定义部署阶段                                 |
| `//`              | 执行部署阶段相关的 一些步骤                  |

## 管道示例

下面是`Jenkinsfile`使用Declarative Pipeline 语法和Scripted Pipeline的示例：

Jenkinsfile (Declarative Pipeline)

```groovy
pipeline { 
    agent any 
    stages {
        stage('Build') { 
            steps { 
                sh 'make' 
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml' 
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```



Jenkinsfile (Scripted Pipeline)

```groovy
node { 
    stage('Build') { 
        sh 'make' 
    }
    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml' 
    }
    stage('Deploy') {
        sh 'make publish'
    }
}
```

| 参数       | 含义                                                         |
| :--------- | :----------------------------------------------------------- |
| `pipeline` | Declarative Pipeline特定语法，定义一个块，包含用于执行整个管道的内容和指令 |
| `agent`    | Declarative Pipeline特定语法，指定Jenkins为整个Pipeline分配执行程序和工作空间 |
| `stage`    | 描述此Pipeline的一个阶段                                     |
| `steps`    | Declarative Pipeline特定语法，描述了stage                    |
| `sh`       | 步骤，执行给定的shell命令                                    |
| `junit`    | 另一个Pipelline步骤，用于聚合测试报告                        |
| `node`     | Scripted Pipeline特定语法，指定了Jenkins在任何可用的代理/节点上执行此Pipeline |

  