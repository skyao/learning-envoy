---
date: 2018-11-07T15:30:00+08:00
title: Discovery相关定义
menu:
  main:
    parent: "xds-overview"
weight: 303
description : "Discovery"
---

在 envoy 项目下的源文件 `api/envoy/api/v2/route/route.proto` 定义有 

- DiscoveryRequest
- DiscoveryResponse
- DeltaDiscoveryRequest
- DeltaDiscoveryResponse
- Resource

### DiscoveryRequest

DiscoveryRequest 用同样的API为给定 envoy 节点请求一组相同类型的版本化资源。

| Field            | Type                | Description                                                  |
| :--------------- | :------------------ | :----------------------------------------------------------- |
| `version_info`   | `string`            | 请求消息中提供的 version_info 使用最近成功处理的响应接收的version_info，或者如果是第一个请求则设置为空。预期在收到响应之后不会发送新请求，直到客户端实例准备好对新配置进行ACK/NACK。 通过分别返回应用的新API配置版本或先前的API配置版本来进行ACK/NACK。 每个type_url（见下文）都有一个与之关联的独立版本。 |
| `node`           | `core.Node`         | 发起请求的节点                                               |
| resource_names   | string[]            | 要订阅的资源列表，例如集群名称列表或路由配置名称列表。 如果为空，则返回API的所有资源。 LDS/CDS期望空resource_names，因为这是Envoy实例的全局发现。 然后，LDS和CDS响应将意味着需要通过EDS / RDS获取许多资源，这些资源将在resource_names中明确枚举。 |
| `type_url`       | `string`            | 正在请求的资源的类型, e.g. “type.googleapis.com/envoy.api.v2.ClusterLoadAssignment”。对于单独的xDS API（如CDS，LDS等）发出的请求中是隐含的，但是对于ADS需要明确设定。 |
| `response_nonce` | `string`            | 对应于DiscoveryResponse的nonce，进行ACK / NACK。 请参阅上面有关version_info和DiscoveryResponse nonce注释的讨论。 如果没有可用的nonce，这可能是空的，例如，在启动时。 |
| `error_detail`   | `google.rpc.Status` | 当前一个DiscoveryResponse无法更新配置时，将填充此选项。 error_details中的message字段提供与失败相关的客户端内部异常。 它仅供手动调试时使用，不保证在客户端版本中提供的字符串是稳定的。 |

### DiscoveryResponse

DiscoveryResponse 提供一组相同类型的版本化资源以响应 DiscoveryRequest。

| Field          | Type                    | Description                                                  |
| :------------- | :---------------------- | :----------------------------------------------------------- |
| `version_info` | `string`                | 响应数据的版本。                                             |
| `resources`    | `google.protobuf.Any[]` | 响应资源。这些资源是有类型的，而且取决于被调用的API。        |
| `type_url`     | `string`                | 资源类型URL。 如果资源非空，则资源的所有消息的 type_url 必须保持一致。 |
| `nonce`        | `string`                | 对于基于gRPC的订阅，nonce提供了一种在后续DiscoveryRequest中显式地ACK特定DiscoveryResponse的方法。 客户端可能已经在此DiscoveryResponse之前在流上向管理服务器发送了其他消息，这些消息在响应发送时未被处理。 nonce允许管理服务器忽略先前版本的任何进一步的DiscoveryRequest，直到带有nonce的DiscoveryRequest。nonce是可选的，对于不是基于stream的xDS实现来说不是必须的。 |



### DeltaDiscoveryRequest

DeltaDiscoveryRequest 和 DeltaDiscoveryResponse 用于 Delta xDS 的新gRPC端点。

使用Delta xDS，DeltaDiscoveryResponses不需要包含跟踪资源的完整快照。 相反，DeltaDiscoveryResponses是xDS客户端状态的差异。 

在Delta XDS中，每个资源版本都有版本，允许以资源粒度跟踪状态。 

xDS Delta 会话始终位于 gRPC 双向流的上下文中。 允许 xDS 服务器跟踪连接到它的 xDS 客户端的状态。

在 Delta xDS 中，nonce字段是必需的，用于将 DeltaDiscoveryResponse 与 DeltaDiscoveryRequest 配对进行 ACK或NACK。 可选地，响应消息级别 system_version_info 仅用于调试目的。

DeltaDiscoveryRequest 可以在3种情况下发送：

1. xDS 双向 gRPC 流中的初始消息
2. 作为对前一个 DeltaDiscoveryResponse 的 ACK 或 NACK 响应

    在这种情况下，response_nonce 设置为 Response 中的 nonce 值。
    
    ACK 或 NACK 由 error_detail 的缺失或存在确定。

3. 来自客户端的自发 DeltaDiscoveryRequest。

    这样做可以动态添加或删除被跟踪的 resource_names 集合中的元素。在这种情况下，response_nonce 必须为空。

| Field            | Type        | Description                                                  |
| :--------------- | :---------- | :----------------------------------------------------------- |
| `node`           | `core.Node` | 响应数据的版本。                                             |
| `type_url`       | `string`    | 正在请求的资源的类型, e.g. “type.googleapis.com/envoy.api.v2.ClusterLoadAssignment”。对于单独的xDS API（如CDS，LDS等）发出的请求中是隐含的，但是对于ADS需要明确设定。 |
| `resource_names_subscribe` | `string[]` | 要添加到跟踪资源列表的资源名称列表。 |
| `resource_names_unsubscribe` | `string[]` | 要从跟踪资源列表中删除的资源名称列表。 |
| `initial_resource_versions` | `map<string, string>` | 当 DeltaDiscoveryRequest 是流中的第一个时，必须填充此map（假设有资源 - 此字段的目的是使session在重新连接的gRPC流中继续，因此不会在session第一个steam中使用）。 key是xDS客户端已知的xDS资源的资源名称。 Map中的值是关联的资源级别版本信息。 |
| `response_nonce` | `string`    | 当 DeltaDiscoveryRequest 是响应前一个 DeltaDiscoveryResponse 的 ACK 或NACK 消息时，response_nonce 必须是 DeltaDiscoveryResponse 中的nonce。 否则必须省略 response_nonce。 |
| `error_detail`   | `google.rpc.Status` | 当前一个DiscoveryResponse无法更新配置时，将填充此选项。 error_details中的message字段提供与失败相关的客户端内部异常。 |

resource_names_subscribe 和 resource_names_unsubscribe 字段的额外说明：

> DeltaDiscoveryRequests 允许客户端在流的上下文中向被跟踪的资源集合添加或删除单个资源。
> 
> resource_names_subscribe 列表中的所有资源名称都将添加到跟踪资源集合中，而 resource_names_unsubscribe 列表中的所有资源名称将从该组跟踪资源中删除。
> 
> 与xDS不同，空的 resource_names_subscribe 或 resource_names_unsubscribe 列表仅表示不会向资源列表添加或删除任何资源。
> 
> xDS 服务器必须为所有跟踪的资源发送更新，但也可以发送客户端尚未订阅的资源的更新。 此行为类似 xDS。
> 
> 可以为所有类型的 DeltaDiscoveryRequests （初始，ACK/NACK或自发）设置这两个字段。

### DeltaDiscoveryResponse

| Field                 | Type         | Description                                                  |
| :-------------------- | :----------- | :----------------------------------------------------------- |
| `system_version_info` | `string`     | 响应数据的版本（用于debug）。                                |
| `resources`           | `Resource[]` | 响应资源。这些资源是有类型的，和 DeltaDiscoveryRequest 中的 type url 匹配。 |
| `removed_resources`   | `string[]`   | 被删除的资源的资源名称，这些资源在xDS客户端也应该被删除。如果要删除的资源不存在则可以忽略。 |
| `nonce`               | `string`     | nonce为DeltaDiscoveryRequests 提供唯一一种引用到DeltaDiscoveryResponse 的方法。 nonce是必需的。 |

Delta 中用到的 Resource 消息：

| Field      | Type                  | Description                               |
| :--------- | :-------------------- | :---------------------------------------- |
| `name`     | `string`              | 资源名，以便和其他相同类型的资源区分      |
| `version`  | `Resource[]`          | 资源级别版本。容许xDS跟踪单个资源的状态。 |
| `resource` | `google.protobuf.Any` | 被跟踪的资源                              |

