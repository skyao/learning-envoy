---
date: 2018-11-07T14:50:00+08:00
title: ADS
description : "介绍Envoy的XDS API中的ADS"
---



ADS请求与其单个 xDS 对应项具有相同的结构，但可以在单个stream上复用多种资源类型。 

DiscoveryRequest / DiscoveryResponse 中的 type_url 提供了足够的信息来恢复 Envoy 实例和管理服务器上的多路复用的单个API。


ADS 的定义在 `api/envoy/service/discovery/v2/ads.proto`。

注意：ADS 是只能用于 gRPC 的API

```protobuf
service AggregatedDiscoveryService {
  rpc StreamAggregatedResources(stream envoy.api.v2.DiscoveryRequest)
      returns (stream envoy.api.v2.DiscoveryResponse) {
  }

  rpc DeltaAggregatedResources(stream envoy.api.v2.DeltaDiscoveryRequest)
      returns (stream envoy.api.v2.DeltaDiscoveryResponse) {
  }
}
```