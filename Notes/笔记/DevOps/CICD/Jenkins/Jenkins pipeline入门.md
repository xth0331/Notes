# Jenkins Pipeline入门

## 定义管道

使用声明式和脚本式Pipeline来描述你的软件交付Pipeline部分。脚本管道以有限的`Groovy`语法编写。

可以通过以下方式创建管道：

- 通过 Blue Ocean - 在Blue Ocean中设置Pipeline项目后，Blue Ocean Ui可以帮助您编写Pipeline的`Jenkinsfile`并提交给源代码管理。
- 通过经典UI - 可以通过经典UI直接在Jenkins输入基本管道。
- 在SCM中 - 可以手动编写`Jenkinsfile`手动编写，可以将其提交到项目的源代码控制存储库。

使用任意方式定义Pipeline的语法都是相同的，但是当Jenkins支持将Pipeline直接输入到经典UI中时，通常认为最好的做法是`Jenkinsfile`,然后Jenkins将直接从源码控制加载。

### 通过Blue Ocean

如果是Jenkins Pipeline新手，Blue Ocean UI 可以帮助设置Pipeline项目，并通过图形管理编辑器自动为您创建和编写Pipeline（Jenkinsfile）。

作为在Blue Ocean中设置Pipeline项目的一部分，Jenkins会为项目的源代码控制存储配置安全且经过适当的身份验证的连接，因此，通过Blue Ocean的Pipeline编辑器对`jenkinsfile`所做的任何更改都会自动保存并提交给源码管理。

### 通过经典UI

使用经典UI创建的 `Jenkinsfile由Jenkins自己存储。

要通过Jenkins经典UI创建基本Pipeline：

  1. 登录Jenkins。
  2. 从Jenkins主页中，单击左上角的`New Item`。
  3. 在“输入一个任务名称”指定新Pipeline的项目名称。
  4. 单击“Pipeline”，然后单击末尾的”确定“打开Pipeline的配置页面。
  5. 单击页面顶部的“Pipeline”选项 卡，下拉到Pipeline部分。
  6. **Definition**部分选择**Pipeline Script**。
  7. 在**Sceipt**部分水Pipeline代码。



### SCM

复杂的Pipeline很难在经典UI脚本文本区域内编写和维护。

为了简化这一过程，你的Pipeline `Jenkinsfile`可以在文本编辑器或**IDE**中编写并提交到源码控制（可选择使用Jenkins将构建的应用程序代码）。然后，Jenlins可以将`Jenkinsfile`的源代码控制作为Pipeline项目构建过程的一部分进行检查，然后继续执行Pipeline。

要将Pipeline项目配置为使用源代码管理中的`Jenkinsfile`：

1. 按照上述定义你的Pipeline过程，从**Definition**字段中选择**SCM**。
2. 从**SCM**字段中，选择包含`Jenkinsfile`的源代码控制存储库的类型。
3. 完成特定存储库管理系统的字段。
4. 在**Script Path**字段中，指定`Jenkinsfile`路径。

更新指定的存储库时，只要使用SCM轮询触发器配置Pipeline，就会触发新的构建。



## 内置文档

Pipeline附带内置文档功能，可以轻松创建不同复杂程度的Pipeline。这个内置文档是根据Jenkins实例中安装的插件自动生成和更新的。

可以在`JENKINS:8080/pipeline-syntax`中找到内置文档，

## Snippet生成器

内置的**Snippet Generator**有助于为各个步骤创建代码，发现插件提供的新步骤，或为特定步骤尝试不同的参数。

**Snippet Generator** 会动态填充Jenkins实例可用的步骤列表。可用步骤取决于安装的插件，这些插件明确公开了在Pipeline中使用的步骤。

要使用代码生成器生成步骤代码段：

1. 导航至`JENKINS:8080/pipeline-syntax`
2. 在**Sample Step**下拉菜单中选择所需的步骤。
3. 对所选步骤的动态填充区域配置所选步骤。
4. 单击 **Generate Pipeline Script** 。

## 全局变量参考

除了仅显示步骤的**Snippet Generator** 之外，Pipeline还提供了内置的**Global Variable Reference** 。它也是由插件动态填充的，但是，不同的是，**Global Variable Reference**仅包含Pipeline或插件提供的变量文档，这些变量可用于Pipeline。

Pipeline默认提供的变量是：

### ENV

可从**Scripted Pipeline**访问的环境变量，例如： `env.PATH`或`env.BUILD_ID`。

### PARAMS

将为Pipeline定义的所有参数公开为只读Map，例如：`params.MY_PARAM_NAME`。

### currentBuild

可用于发现有关当前正在执行的管道的信息，其中包含``currentBuild.result`，`currentBuild.displayName`等属性。请参阅内置的全局变量参考以获取`currentBuild`上可用的完整且最新的属性列表。

## 声明式指令生成器

虽然Snippet Generator有助于为Scripted Pipeline或Declarative Pipeline中的`steps`块生成步骤`stage`，但它不包括用于定义Declarative Pipeline 的部分和指令。“声明指令生成器”实用程序有助于此。与Snippet Generator类似，Directive Generator允许您选择Declarative指令，在表单中配置它，并为该指令生成配置，然后您可以在Declarative Pipeline中使用该配置。

使用Declarative Directive Generator生成Declarative指令：

1. 从配置的管道导航到**Pipeline Syntax**链接（如上所述），然后单击sidepanel中的**Declarative Directive Generator**链接，或直接转到[localhost：8080 / directive-generator](http://localhost:8080/directive-generator)。
2. 在下拉菜单中选择所需的指令
3. 使用下拉列表下方的动态填充区域配置所选指令。
4. 单击**生成指令**以创建指令的配置以复制到管道中。

指令生成器可以为嵌套指令生成配置，例如`when`指令内的条件，但它不能生成管道步骤。对于包含步骤的指令的内容，例如`steps`insde a `stage`或类似`always`或`failure`内部的条件`post`，指令生成器会添加占位符注释。您仍然需要手动向管道添加步骤。



Jenkinsfile (Declarative Pipeline)

```groovy
stage('Stage 1') {
    steps {
        // One or more steps need to be included within the steps block.
    }
}
```