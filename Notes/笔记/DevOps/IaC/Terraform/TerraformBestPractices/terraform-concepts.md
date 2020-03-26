# Terraform 关键概念

## Resource



`Resource`属于`provider`，接受参数，输出属性，具有生命周期。可以创建，检索，更新和删除`resource`.

## Resource module



## 基础架构模块（Infrastucture module）

基础架构模块是资源模块的集合，这些资源模块在逻辑上可以不连接，但是在当前情况、项目、设置中可以达到相同的目的。它定义了提供程序的配置，该配置被传递到下游资源模块和资源。通常限制为每个逻辑分隔符只能在一个实体中工作。

## Composition

Composition是基础架构模块的集合，可以跨几个逻辑上分开的区域。组合用于描述整个Composition或者项目所需的完整基础架构。

Composition由基础架构模块组成，基础架构模块由实现单个资源模块组成。

![简单的基础架构组成](https://blog-image.nos-eastchina1.126.net/terraform01.png)





## Data soutce

`Data source`执行只读操作，并且取决于提供程序，他在资源模块和基础架构模块中使用。

数据源 `terraform_remote_state`充当高级模块和组合的粘合剂。

外部数据源允许外部程序充当数据源，公开任意数据以供terraform配置中的其他地方使用。

该数据源发出HTTP GET请求，其中本地Terraform提供者不存在信息的相应给定的URL和出口信息。



## Remote state

基础架构模块和组合应该讲其状态保存在远程位置，其他人可以通过可控制的方式访问（ACL，版本控制，日志记录）它们。

## Provider, provisioner

和其他一些术语在官方文档中都有很好的描述，在我看来，它们与编写良好的terraform模块无关。

## 为什么这么难

尽管单个资源就像基础架构中的原子，但资源模块却是分子。模块的最小版本化和可共享的单元。它具有确切的参数列表，为该单元实现所需功能实现基本逻辑。资源模块本身可以与其他模块一起使用，以创建基础架构模块。

使用模块输出和数据源执行跨分子（资源模块和基础架构模块）的数据访问。

组合之间的访问时使用远程状态数据源执行的。

将上述概念放入伪关系中时，可能开起来像这样：

```bash
composition-1 {
  infrastructure-module-1 {
    data-source-1 => d1

    resource-module-1 {
      data-source-2 => d2
      resource-1 (d1, d2)
      resource-2 (d2)
    }

    resource-module-2 {
      data-source-3 => d3
      resource-3 (d1, d3)
      resource-4 (d3)
    }
  }

}
```

