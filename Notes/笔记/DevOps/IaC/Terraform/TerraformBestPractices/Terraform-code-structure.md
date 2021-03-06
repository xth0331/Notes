# Terraform代码结构

## 应该如何构建Terraform配置？

这是存在许多解决方案的问题之一，很难获得一般性建议，因此首先了解我们正在处理的问题。

- 项目复杂性？
  - 相关资源数量
  - Terraform提供者的数量
- 基础架构多久变更一次？
  - 从每月/每周/每天/一次
  - 持续不断
- 代码变发起者? *是否让CI服务器更新存储库*
  - 只有开发人员可以push到基础架构存储库
  - 每个人都可以通过打开PR（包括在CI服务器上运行的自动化任务）来对任何内容进行变更
- 使用哪个部署平台部署服务
  - AWS CodeDeploy， Kubernetes或OpenShift
- 环境如何分组
  - 按环境，区域，项目划分



## Terraform配置结构入门

开始或编写示例代码时，最好将所有代码都放入`main.tf`其中。在所有其他情况下，最好将多个文件按如下逻辑进行拆分。

- `main.tf` 