# Kubernetes对象之names



Kubernetes REST API中的所有对象都由Name和UID明确标识。

对于非唯一的用户提供的属性，Kubernetes系统提供`label`(标签)和注释。

## Names

客户端提供的字符串，用于引用资源URL中的对象，例如`/api/v1/pods/some-name`。

只有给定类型的一个对象一次可以有一个给定的名称。但是，如果删除对象，则可以创建有相同名称的新对象。

按照惯例，Kubernetes资源的名称应最多为253个字符，并且由小写字母数字字符`-`和`.`组成。



## UIDs

Kubernetes系统生成的字符串，用于唯一标识对象。

在Kubernetes集群的整个生命周期中创建的每个对象都具有不同的UID，它旨在去跟类似实体的历史事件。

