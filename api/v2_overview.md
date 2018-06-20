# Envoy v2 API概述

> 翻译自：https://github.com/envoyproxy/data-plane-api/blob/master/API_OVERVIEW.md

## 目标

该存储仓库包含已实现的和草案版本的v2 JSON REST和gRPC Envoy API。

Envoy API的第2版是现有的API的演进，并引入了新的API，以便：

- 通过向管理服务器报告负载和资源利用率，容许更高级的负载平衡。
- 通过选择性地将健康检查卸载到其他Envoy实例来改善N^2健康检查可伸缩性问题。
- 通过对监听器模型和CDS/SDS API的更改来支持边缘，sidecar和中间代理部署模型中的Envoy部署。
- 发生变更时，容许从管理服务器上的获取流式更新，替代从Envoy发起轮询API。gRPC API将与JSON REST API一起提供支持。
- 确保所有Envoy运行时配置均可通过API调用动态发现，其中包括监听器配置，证书和运行时设置，这些设置现在都来自文件系统。仍然会保留一个静态引导配置文件，该文件将设定在运行时不可能更改的项目，包括Envoy节点标识，xDS管理服务器地址，管理界面和追踪配置。
- 回顾并在适当的情况下清理任何v1的技术债务。

## 状态

请参阅[此处](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v2_overview.html#status)了解v2 API的当前状态。

请参阅[此处](https://github.com/envoyproxy/data-plane-api/blob/master/CONTRIBUTING.md#api-changes)以了解v2 API更改过程。

## 原则

- [Proto3](https://developers.google.com/protocol-buffers/docs/proto3) 将用于指定规范API。这将直接提供gRPC API并通过gRPC-JSON转码为JSON REST API。作为语法的便利，除了JSON之外，还将支持文件系统配置文件（例如引导文件）的文本YAML输入。YAML文件内容将在内部转换为JSON，然后在Envoy配置获取期间遵循标准JSON-proto3转换。
- xDS API应该支持最终一致性。例如，如果RDS引用尚未由CDS提供的集群，则应静默地忽略它并且不转发流量，直到CDS更新发生。如果管理服务器能够仔细排序xDS API（例如，使用下面的ADS API），则可以实现更强的一致性保证。通过遵循所有相关资源的`[CDS，EDS，LDS，RDS]`顺序，可以避免配置更新期间的流量中断。
- 该API主要用于机器生成和消费。预计管理服务器负责将更高级别的配置概念映射到API响应。同样，静态配置片段可能由模板工具等生成。用于生成xDS配置的API和工具超出了此存储仓库中定义的范围。


- 将为基础单例xDS订阅服务CDS/EDS/LDS/RDS/SDS提供等效的REST-JSON API。HDS，ADS和EDS多维LB等高级API仅适用于gRPC。避免将复杂的双向流语义映射到REST上。
- 监听器将不可变。通过LDS对监听器进行的任何更新都要求逐出指定绑定IP/端口的现有连接。因此，只有在现有连接耗尽或耗尽超时后才能保证新请求将遵守新配置。
- 版本控制将通过[proto3包命名空间](https://developers.google.com/protocol-buffers/docs/proto3#packages)表示，即软件包envoy.api.v2 ;
- 定制组件（例如过滤器，解析器，记录器）将使用反向DNS命名方案，例如， com.google.widget，com.lyft.widget。
- Versioning will be expressed via [proto3 package namespaces](https://developers.google.com/protocol-buffers/docs/proto3#packages), i.e. `package envoy.api.v2;`.
- 定制化组件 (如 filters, resolvers, loggers) 将使用反向 DNS 命名方案，如 `com.google.widget`, `com.lyft.widget`.

## APIs

除非另有说明，与v1 API相同名称的API具有类似的作用。

- [Cluster Discovery Service (CDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/cds.proto).
- [Endpoint Discovery Service (EDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/eds.proto). 这与v1 API中的SDS具有相同的作用，新名称更好地描述了API在实践中的作用。能够利用N维上游指标的高级全局负载均衡现在可以支持了。
- [Health Discovery Service (HDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/service/discovery/v2/hds.proto). This new API supports efficient endpoint health discovery by the management server via the Envoy instances it manages. Individual Envoy instances will typically receive HDS instructions to health check a subset of all endpoints. The health check subset may not be a subset of the Envoy instance's EDS endpoints. 此新API支持管理服务器通过其管理的Envoy实例进行高效的端点健康发现。单独的Envoy实例通常会接收HDS指令以对所有端点的子集进行健康检查。运行健康检查子集可能不是Envoy实例的EDS端点的子集。
- [Listener Discovery Service (LDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/lds.proto). 这个新API支持动态发现监听器配置（绑定哪些端口，TLS细节，过滤器链等）。
- [Metric Service (MS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/service/metrics/v2/metrics_service.proto). 这个新的API允许Envoy一直推送（流）指标，以供服务器使用。
- [Rate Limit Service (RLS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/service/ratelimit/v2/rls.proto)
- [Route Discovery Service (RDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/rds.proto).
- [Secret Discovery Service (SDS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/service/discovery/v2/sds.proto).

除了上述API之外，还将提供一个聚合API，以便对跨发现服务的API更新排序进行细粒度控制：

- [Aggregated Discovery Service (ADS)](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/api/v2/discovery.proto). 看 [ADS 概述](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v2_overview#aggregated-discovery-service).

xDS API的protocol描述在 [这里](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md) 提供。

## 术语

下面重复一些相关的[现有术语](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/terminology.html)，并引入一些新的v2术语。

- 集群/Cluster: 集群是指 Envoy 连接到的逻辑上相同的一组上游主机。在v2中，RDS路由指向集群，CDS提供集群配置，Envoy通过EDS发现集群成员。
- 下游/Downstream：下游主机连接到 Envoy，发送请求并接收响应。
- 端点/Endpoint：端点是上游主机，它是一个或多个集群的成员。端点通过EDS发现。
- 监听器/Listener: 监听器是命名网络地址（例如，端口、unix domain socket等)，可以被下游客户端连接。Envoy 暴露一个或者多个监听器给下游主机连接。
- 局部性/Locality：Envoy实例或端点运行的位置。这包括区域/region，区域/zone和子区域/sub-zone标识。
- 管理服务器：实现v2 Envoy API的逻辑服务器。这不一定是单个物理机器，因为它可以被复制/分片，并且用于不同xDS API的API服务可以在不同的物理机器上实现。
- 区域/region：区域所在的地理区域。
- 子区域/Sub-zone：Envoy实例或端点运行的区域内的位置。这允许区域内的多个负载均衡目标。
- 区域/Zone: AWS中的Availability Zone (AZ), GCP中的Zone
- 上游/Upstream: 上游主机接收来自 Envoy 的连接和请求，并返回响应。
- xDS: CDS/EDS/HDS/LDS/RLS/RDS/SDS APIs.

