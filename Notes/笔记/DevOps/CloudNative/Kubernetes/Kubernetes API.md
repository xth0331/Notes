# Kubernetes API

Kubernetes API可用作系统声明式配置架构的基础。`kubectl`命令行工具可以用来创建，更新，删除，并获得API对象。

Kubernetes还提供API资源存储其序列化状态（ETCD）。

Kubernetes本身被分解为多个组件，通过其API进行交互。

- API变更
- OpenAPI和Swagger定义
- API版本控制
- API groups
- 启用API groups
- 启用组中的资源



## API changes

根据经验，任何成功的系统都需要随着新用例的出现或用例的变化而增长和变化。但是，一段时间内不破坏与现有客户端的兼容性。通常可以预期频繁添加新的API资源和新的资源字段。

## OpenAPI and Swagger definitions

使用OpenAPI记录完整的API详细信息。

从Kubernetes 1.10开始， Kubernetes API server 通过`/openapi/v2` 端点提供OpenAPI规范。 通过设置 HTTP headers指定请求的格式:

| Header          | Possible Values                                              |
| --------------- | ------------------------------------------------------------ |
| Accept          | `application/json`, `application/com.github.proto-openapi.spec.v2@v1.0+protobuf` (the default content-type is `application/json` for `*/*` or not passing this header) |
| Accept-Encoding | `gzip` (not passing this header is acceptable)               |

Prior to 1.14, format-separated endpoints (`/swagger.json`, `/swagger-2.0.0.json`, `/swagger-2.0.0.pb-v1`, `/swagger-2.0.0.pb-v1.gz`) serve the OpenAPI spec in different formats. These endpoints are deprecated, and will be removed in Kubernetes 1.14.

**Examples of getting OpenAPI spec**:

| Before 1.10                 | Starting with Kubernetes 1.10                                |
| --------------------------- | ------------------------------------------------------------ |
| GET /swagger.json           | GET /openapi/v2 **Accept**: application/json                 |
| GET /swagger-2.0.0.pb-v1    | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf |
| GET /swagger-2.0.0.pb-v1.gz | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf **Accept-Encoding**: gzip |

Kubernetes为API实现了另一种基于Protobuf的序列化格式，主要用于集群内通信，在设计提案中有记录，每个模式的IDL文件都位于定义API对象的Go包中。

在1.14之前，Kubernetes apiserver 还公开了一个API，可用于检索`/swaggerapi`中的 Swagger v1.2 Kubernetes API 规范 .将在 Kubernetes 1.14中删除.

## API versioning

为了更容易消除资源或重构资源表示，Kubernetes支持多个API版本，每个API版本位于不同的API路径，例如 `/api/v1` 或 `/apis/extensions/v1beta1`。

我们选择在API级别而不是在资源或字段级别进行版本化，以确保API提供清晰，已知的系统资源和行为视图，并允许控制对生命周期末端和/或实验API的访问。 JSON和Protobuf序列化模式遵循相同的模式更改指南——以下所有描述都涵盖两种模式。

请注意，API版本控制和软件版本控制仅间接相关。API和发布版本控制提议描述了API版本控制和软件版本控制之间的关系。

不同的API版本意味着不同级别的稳定性和支持。API更改文档中更详细地描述了每个级别的标准。总结如下:

- Alpha level:
  - 版本名包含 `alpha` (e.g. `v1alpha1`).
  - 启用该功能可能会暴露错误。默认禁用。
  - 可随时删除对功能的支持，不另行通知。
  - API可能会在以后的软件版本中以不兼容方式更改。
  - 由于错误风险增加和缺乏长期支持，建议尽在短期测试集群中使用。
- Beta level:
  - 版本名称包含 `beta` (e.g. `v2beta3`).
  - 代码经过充分测试。默认该功能被认为是安全的。默认情况启用。
  - 虽然细节可能会有所变化，但不会删除对整体功能的支持。
  - 在随后的beta版或stable版，对象的模式和/或语义可能以不兼容的方式发生变化。发生这种情况时，可能需要删除，编辑和重新创建API对象。编辑过程可能需要一些思考。对于依赖该功能的应用程序，可能需要停机时间。
  - 建议仅用于非关键业务用于，因为后续版本中可能存在不兼容的更改。如果有富哦个可以独立升级的集群，可以放宽此限制。
- Stable level:
  - 版本名称是 `vX` ， `X` 是一个整数。
  - 许多后续版本的已发布软件中将出现稳定版本的功能。

## API groups

为了更容易扩展Kubernetes API，我们实现了API groups。API groups在REST路径和apiVersion序列化对象的字段中指定。

目前有几个API groups正在使用:

1.  *core* group, 常被称为 *legacy group*, 位于 REST 路径 `/api/v1`并使用 `apiVersion: v1`。
2. named groups 位于 REST 路径 `/apis/$GROUP_NAME/$VERSION`, 并使用 `apiVersion: $GROUP_NAME/$VERSION` (例如 `apiVersion: batch/v1`)。

使用自定义资源扩展API有两种受支持的路径:

1. CustomResourceDefinition 适用于具有非常基本 CRUD 需求的用户。
2. 需要全套Kubernetes API语义的用户可以实现自己apiserver并使用 `aggregator`使其无缝地为客户端。

## Enabling API groups

默认情况下启用某些资源和API groups。可以通过在apiserver上设置 `--runtime-config`来启用或禁用他们。 `--runtime-config` 接受逗号分隔的值。例如要禁用batch/v1, 请设置 `--runtime-config=batch/v1=false`, 要启用 batch/v2alpha1, 请设置`--runtime-config=batch/v2alpha1`.该标志接受逗号分隔的一组`key=value`键值对，描述了apiserver的运行时配置。.

要启用或禁用组或资源需要重新启动`apiserver`和`controller-manager`以生效 `--runtime-config`更改.

## Enabling resources in the groups

默认情况下启用DaemonSets, Deployments, HorizontalPodAutoscalers, Ingresses, Jobs and ReplicaSets . 可以通过在`apiserver`上设置 `--runtime-config` 来启用其他扩展资源. `--runtime-config` 接受逗号分隔. 例如要禁用: to disable deployments and ingress, 设置 `--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingresses=false`

