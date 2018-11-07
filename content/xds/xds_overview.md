---
date: 2018-11-07T15:30:00+08:00
title: xDS REST和gRPC协议
menu:
  main:
    parent: "xds"
weight: 301
description : "xDS REST和gRPC协议"
---

> 备注：内容来自 https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md

Envoy通过文件系统或通过查询一个或多个管理服务器来发现其各种动态资源。这些发现服务及其相应的API统称为xDS。 通过订阅，指定要监视的文件系统路径，启动gRPC流或轮询REST-JSON URL来请求资源。后两种方法涉及使用DiscoveryRequest proto 载荷发送请求。在所有方法中资源以DiscoveryResponse proto 负载的形式发送。我们在下面讨论每种类型的订阅。

## 文件系统订阅

提供动态配置的最简单方法是将其放置在 [ConfigSource](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/config_source.proto#core-configsource) 中指定的众所周知的路径中。Envoy将使用`inotify`（Mac OS X上的kqueue）来监视文件的更改，并在更新时解析文件中的`DiscoveryResponse` proto。二进制protobufs，JSON，YAML和proto文本是`DiscoveryResponse`支持的格式。

除了统计计数器和日志以外，没有任何机制可用于文件系统订阅ACK/NACK更新。如果发生配置更新拒绝，xDS API的最后一个有效配置将继续适用。

## 流式gRPC订阅

### 单体资源类型发现

可以为每个xDS API独立指定gRPC  [ApiConfigSource](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/config_source.proto#core-apiconfigsource)，指向与管理服务器对应的上游集群。这将为每个xDS资源类型启动一个独立的双向gRPC流，可能会发送给不同的管理服务器。 API发送是最终一致。请参阅下面的ADS，了解需要明确控制顺序的情况。

#### 类型URL

每个xDS API都关注于给定类型的资源。 xDS API和资源类型之间存在1：1的对应关系。如：

- [LDS: `envoy.api.v2.Listener`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto)
- [RDS: `envoy.api.v2.RouteConfiguration`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto)
- [CDS: `envoy.api.v2.Cluster`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto)
- [EDS: `envoy.api.v2.ClusterLoadAssignment`](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto)
- [SDS: envoy.api.v2.Auth.Secret](https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/auth/cert.proto)

[*类型 URLs*](https://developers.google.com/protocol-buffers/docs/proto3#any) 的概念显示在下方，其格式为 `type.googleapis.com/<resource type>`，例如 用于CDS的`type.googleapis.com/envoy.api.v2.Cluster`。 在来自Envoy的各种请求和管理服务器的响应中，说明了资源类型URL。

#### ACK/NACK and versioning

每个流都以Envoy的 `DiscoveryRequest` 开始，指定要订阅的资源列表，与订阅的资源对应的类型URL，节点标识符和空的`version_info`。 示例EDS请求可能是：

```ymal
version_info:
node: { id: envoy }
resource_names:
- foo
- bar
type_url: type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
response_nonce:
```

管理服务器可以立即回复，或者当请求的资源可用时回复，使用`DiscoveryResponse`，例如：

```yaml
version_info: X
resources:
- foo ClusterLoadAssignment proto encoding
- bar ClusterLoadAssignment proto encoding
type_url: type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
nonce: A
```

在处理`DiscoveryResponse`之后，Envoy将在流上发送新的请求，指定成功应用的最后一个版本和管理服务器提供的随机数。 如果更新已成功应用，`version_info`将为X，如序列图中所示：

![Version update after ACK](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/simple-ack.svg?sanitize=true)

在此序列图及其下方，使用以下格式缩写消息：

- `DiscoveryRequest`: (V=`version_info`,R=`resource_names`,N=`response_nonce`,T=`type_url`)
- `DiscoveryResponse`: (V=`version_info`,R=`resources`,N=`nonce`,T=`type_url`)

`version`为Envoy和管理服务器提供了当前已应用配置的共享概念，以及ACK/NACK配置更新的机制。如果Envoy拒绝配置更新X，它将回复 [error_detail](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/discovery.proto#envoy-api-field-discoveryrequest-error-detail) 和其以前的版本，在这种情况下，它是空的初始版本。 error_detail在消息字段中填充了确切的错误消息的详细信息：

![No version update after NACK](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/simple-nack.svg?sanitize=true)

稍后，API更新可能成功为新版本Y：

![ACK after NACK](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/later-ack.svg?sanitize=true)

每个流都有自己的版本控制概念，没有跨资源类型的共享版本。当不使用ADS时，由于Envoy API允许不同的EDS/RDS资源指向不同的ConfigSource，因此即使给定资源类型的每个资源也可能具有不同的版本。

#### 何时发送更新

仅当`DiscoveryResponse`中的资源发生更改时，管理服务器会向Envoy客户端发送更新。在被接受或拒绝之后，Envoy立即使用包含ACK/NACK的`DiscoveryRequest` 回复任何 `DiscoveryResponse`。如果管理服务器提供相同的一组资源而不是等待发生更改，则会导致Envoy和管理服务器陷入循环并对性能造成严重影响。

在一个流内，新的`DiscoveryRequests`取代了具有相同资源类型的任何先前的`DiscoveryRequest`。这意味着管理服务器只需响应任何给定资源类型的每个流上的最新`DiscoveryRequest`。

#### 资源提示

`DiscoveryRequest`中指定的`resource_name`是提示。一些资源类型，例如`Cluster`和 `Listener` 将指定一个空的`resource_names`列表，因为Envoy有兴趣了解管理服务器知道的与集群节点标识相对应的所有集群（CDS）和监听器（LDS）。其他资源类型，例如`RouteConfigurations`（RDS）和`ClusterLoadAssignments`（EDS），从早期的CDS/LDS更新开始，Envoy能够明确地列举这些资源。

LDS/CDS资源提示将始终为空，并且预计管理服务器将在每个响应中提供完整状态的LDS/CDS资源。缺席的监听器或集群将被删除。

对于EDS/RDS，管理服务器不需要提供每个请求的资源，并且还可以提供额外的未请求的资源，`resource_names`只是一个提示。Envoy将默默地忽略任何多余的资源。当RDS或EDS更新中缺少请求的资源时，Envoy将保留此资源的最后一个已知值。管理服务器可能能够从发现请求中的 `node` 标识推断出所有需要的EDS/RDS资源，在这种情况下，该提示可能被丢弃。从Envoy各自资源的角度来看，空的EDS/RDS `DiscoveryResponse`实际上是一个`nop`。

当 `Listener` 或者 `Cluster` 被删除时，其相应的EDS和RDS资源也会在Envoy实例内被删除。为了让Envoy知道或跟踪EDS资源，必须存在应用的集群定义（例如，来源于CDS）。RDS和 `Listeners` 之间存在类似的关系（例如，来源于LDS）。

对于EDS/RDS，Envoy可以为给定类型的每个资源生成一个不同的流（例如，如果每个`ConfigSource`都有自己独立的管理服务器的上游集群），或者可以为给定资源类型合并多个资源请求，当他们用的管理服务器相同时。这留给实现细节，管理服务器应该能够处理每个请求中给定资源类型的一个或多个`resource_name`。以下两个序列图均适用于获取两个EDS资源 `{foo，bar}`：

![Multiple EDS requests on the same stream](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/eds-same-stream.svg?sanitize=true)

![Multiple EDS requests on distinct streams](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/eds-distinct-stream.svg?sanitize=true)

#### 资源更新

如上所述，Envoy可能会在每个`DiscoveryRequest`中更新其呈现给管理服务器的`resource_names`列表，以确认/否定特定的`DiscoveryResponse`。另外，Envoy稍后可以在给定的`version_info`上发起额外的 `DiscoveryRequest`，以使用新的资源提示来更新管理服务器。例如，如果Envoy处于EDS版本X并且只知道cluster foo，但是接收到CDS更新并了解有关bar的信息，则可以使用`{foo，bar}`作为`resource_names`为X另外发出一个`DiscoveryRequest`。

![CDS response leads to EDS resource hint update](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/cds-eds-resources.svg?sanitize=true)

这里可能会出现一种竞争条件; 如果在X处由Envoy发起资源提示更新之后，但在管理服务器处理更新之前，它使用了新版本Y回复，则资源提示更新可能会被解读为通过呈现X版本信息来拒绝Y. 为避免这种情况，管理服务器提供Envoy用来指示每个DiscoveryRequest对应的特定DiscoveryResponse的随机数：

![EDS update race motivates nonces](https://github.com/envoyproxy/data-plane-api/raw/master/diagrams/update-race.svg?sanitize=true)

管理服务器不应为任何具有过期nonce的DiscoveryRequest发送DiscoveryResponse。在DiscoveryResponse中向Envoy提交新的nonce之后，nonce变得过期。管理服务器在确定新版本可用之前不需要发送更新。一个版本的早期请求也变得过期。它可以在一个版本上处理多个DiscoveryRequests，直到新版本准备就绪。

![](https://github.com/envoyproxy/envoy/raw/master/api/diagrams/stale-requests.svg?sanitize=true)

上述资源更新排序的一个含义是Envoy不期望它发出的每个DiscoveryRequest都有DiscoveryResponse。

#### 最终一致性考虑因素

由于Envoy的xDS API是最终一致性的，因此在更新期间流量可能会短暂下降。例如，如果通过CDS/EDS只知道集群X，则RouteConfiguration会引用集群X，然后在提供Y的CDS/EDS更新之前调整为集群Y，流量将被无法处理，直到Envoy实例知道Y.

对于某些应用程序，可以接受临时的流量下降，客户端或其他Envoy sidecar的重试将隐藏此下降。对于无法容忍丢弃的其他情况，可以通过下列方式来避免流量下降：提供包括X和Y的CDS/EDS更新，然后RDS更新从X重新分配到Y，然后CDS/EDS更新丢弃X.

通常，为了避免流量下降，更新的顺序应遵循`make before break`模型，其中：

- CDS更新（如果有）必须始终优先推送。
- EDS更新（如果有）必须在相应群集的CDS更新后到达。
- LDS更新必须在相应的CDS/EDS更新后到达。
- 与新添加的侦听器相关的RDS更新必须最终到达。
- 然后可以删除过时的CDS群集和相关的EDS端点（不再被引用的端点）。

如果未添加新的群集/路由/侦听器，或者可以接受在更新期间临时丢弃流量，则可以单独推送xDS更新。请注意，在LDS更新的情况下，侦听器将在接收流量之前进行热身，即如果已配置，则通过RDS获取相关路由。 添加/删除/更新群集时，群集会热身。另一方面，路由不做热身，即，在推送路由的更新之前，管理平面必须确保路由引用的集群就位。

### 聚合发现服务（ADS）

当管理服务器是分布式部署时，在排序时提供上述保证以避免分发时的流量跌落是非常具有挑战性的。ADS允许单个管理服务器通过单个gRPC流提供所有API更新。这提供了仔细排序更新以避免流量下降的能力。使用ADS，单个流与通过类型URL多路复用的多个独立的DiscoveryRequest/DiscoveryResponse序列一起使用。 对于任何给定类型的URL，上述DiscoveryRequest和DiscoveryResponse消息的顺序均适用。 示例更新序列可能如下所示：

![](https://github.com/envoyproxy/envoy/raw/master/api/diagrams/ads.svg?sanitize=true)

每个Envoy实例都可以使用单个ADS流。

ADS配置的最小bootstrap.yaml片段示例如下：

```yaml
node:
  id: <node identifier>
dynamic_resources:
  cds_config: {ads: {}}
  lds_config: {ads: {}}
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: ads_cluster
static_resources:
  clusters:
  - name: ads_cluster
    connect_timeout: { seconds: 5 }
    type: STATIC
    hosts:
    - socket_address:
        address: <ADS management server IP address>
        port_value: <ADS management server port>
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
admin:
  ...
```

### 增量 xDS

增量 xDS是可用于ADS，CDS和RDS的单独xDS端点，允许:

- xDS客户端对跟踪资源列表的增量更新。

	这将支持Envoy按需/延迟请求额外资源。例如，当与未知集群相对应的请求到达时，可能会发生这种情况。

- xDS服务器可以逐步更新客户端上的资源。

	这将支持xDS资源可伸缩性的目标。管理服务器只需发送更改的单个集群，而不是在修改单个集群时交付所有100k集群。

xDS增量会话始终位于gRPC双向流的上下文中。 这允许xDS服务器跟踪连接到它的xDS客户端的状态。 Incremental xDS没有REST版本。

在增量 xDS 中，nonce 字段是必须的，用于将 IncrementalDiscoveryResponse 匹配到 IncrementalDiscoveryRequest ACK 或者 NACK。可选的，应答消息级别的 system_version_info 仅用于调试目的。

`IncrementalDiscoveryRequest`可以在以下 3 中情况下发送：
  
1. xDS 双向 gRPC 流中的初始化信息。

2. 作为前一个`IncrementalDiscoveryResponse`的ACK或者NACK应答

	在这种情况下， `response_nonce` 被设置为应答中的nonce值。
	
	ACK或者NACK通过`error_detail`的存在与否进行判断。
	
3. 客户端自发的 `IncrementalDiscoveryRequest` 

	可以这样做以从跟踪的 `resource_names` 集合中动态添加或删除元素。 在这种情况下，必须省略 `response_nonce`。

在第一个示例中，客户端连接并接收第一个更新，并进行ACK确认。第二次更新失败，客户端NACK更新。之后xDS客户端自发地请求“wc”资源。

![Incremental session example](https://github.com/envoyproxy/envoy/raw/master/api/diagrams/incremental.svg?sanitize=true)

On reconnect the xDS Incremental client may tell the server of its known resources
to avoid resending them over the network.

重新连接时，xDS Incremental客户端可能会告知服务器其已知资源，以避免通过网络重新发送它们。

![Incremental reconnect example](https://github.com/envoyproxy/envoy/raw/master/api/diagrams/incremental-reconnect.svg?sanitize=true)

## REST-JSON 轮训订阅

通过REST端点进行的同步（长）轮询也可用于xDS单例API。上面的消息顺序是类似的，除了没有维护到管理服务器的持久流。 预计在任何时间点只有一个未完成的请求，因此响应nonce在REST-JSON中是可选的。proto3的JSON规范转换用于编码DiscoveryRequest和DiscoveryResponse消息。 ADS不适用于REST-JSON轮询。

当轮询周期设置为较小的值时，为了进行长轮询，则还需要避免发送DiscoveryResponse，除非发生了对底层资源的更改。
