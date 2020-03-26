# Terraform 配置语言

Terraform旨在对基础架构进行简介的描述。Terraform配置语言是声明式的，描述是预期目标，而不是达到目标的步骤。

## 资源和模块

Terraform配置语言的主要目的是声明资源。所有其他语言功能的存在只是为了使资源的定义更加灵活和方便。

可以将一组资源收集到一个模块中，该模块创建一个更大的配置单元。资源描述单个基础架构对象，而模块可能描述一组对象以及它们之间的必要关系，以便创建更高级别的系统。

*一个Terraform配置由一个根模块组成*，随着一当一个模块调用另一个模块的子模块。

## 参数，块和表达式

Terraform语言和语法仅包含一些基本元素：

```yaml
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

-  `block`是内容的容器，通常代表某种对象的配置。`block`具有类型，可以具有零个或多个标签，并且具有包含任意数量的参数喝嵌套`block`主体。Terraform的大多数功能由配置文件仲的顶级块控制。
- 参数为名称分配一个值。它们出现在块内。
- 表达式可以字面地表示活通过引用并且组合其他值来表示一个值。它们显示为参数值或其他表达式中的值。

## 代码结构

Terraform语言使用以`.tf`文件拓展名命名的配置文件。还有一种基于`JSON`的变体，其文件扩展名为`tf.json` 。

配置文件必须始终使用`UTF-8`编码，并且按照惯例，通常使用`Unix`样式的行尾（LF）而不是Windows样式行尾（CRLF）进行维护，尽管两者都被接受。

模块是保存在目录中的`.tf`或`.json.tf`的集合。运行Terraform时，根模块是根据当前工作目录中的配置文件构建的，该模块可以引用其他目标中的子模块，而这些子模块又可以引用其他模块等。

最简单的Terraform配置是仅包含单个`.tf`文件的单个根模块。随着添加更多资源，配置可以逐渐增长，方法是在根模块中创建新的配置文件，或者将资源集组组织到子模块中。

> 因为Terraform的配置语言是声明式的，所以块的顺序通常并不重要（`provisioner`资源块的顺序会影响）
>
> Terraform根据配置中定义的资源之间的关系自动以正确的顺序处理资源，因此可以采用对基础架构有意义的任何方式将资源组织到源文件中。

## Terraform CLI与Providers

Terraform命令行界面（CLI）是用于评估和应用Terraform配置的通用引擎。它定义了Terraform语言的语法和整体架构，并协调了使远程基础架构与给定配置所必须进行的更改序列。

该通用引擎不了解特定类型的基础架构对象。相反，Terraform使用称为`providers` 插件，每个插件都定义和惯例一组资源类型。

大多数`providers`都与特定的云或本地基础架构服务相关联，从而使Terraform可以管理该服务中的基础结构对象。

Terraform没有与平台无关的资源类型的概念-资源始终与`providers`绑定，因为不同`providers`之间相似资源的功能可能会有很大差异。但是Terraform CLI的共享配置引擎确保所有服务都可以使用相同的构建语法，并允许根据需要组合来自不同服务的资源类型。

## 范例

以下简单示例描述了Amazon Web Services的简单网络拓扑，只是为了大致了解Terraform语言的整体结构和语法。可以使用其他提供商定义的资源类型为其他虚拟网络服务创建类似的配置，而实际的网络配置通常会包含此处未显示的其他元素。

```yaml
variable "aws_region" {}

variable "base_cidr_block" {
  description = "A /16 CIDR range definition, such as 10.1.0.0/16, that the VPC will use"
  default = "10.1.0.0/16"
}

variable "availability_zones" {
  description = "A list of availability zones in which to create subnets"
  type = list(string)
}

provider "aws" {
  region = var.aws_region
}

resource "aws_vpc" "main" {
  # Referencing the base_cidr_block variable allows the network address
  # to be changed without modifying the configuration.
  cidr_block = var.base_cidr_block
}

resource "aws_subnet" "az" {
  # Create one subnet for each given availability zone.
  count = length(var.availability_zones)

  # For each subnet, use one of the specified availability zones.
  availability_zone = var.availability_zones[count.index]

  # By referencing the aws_vpc.main object, Terraform knows that the subnet
  # must be created only after the VPC is created.
  vpc_id = aws_vpc.main.id

  # Built-in functions and operators can be used for simple transformations of
  # values, such as computing a subnet address. Here we create a /20 prefix for
  # each subnet, using consecutive addresses for each availability zone,
  # such as 10.1.16.0/20 .
  cidr_block = cidrsubnet(aws_vpc.main.cidr_block, 4, count.index+1)
}
```

