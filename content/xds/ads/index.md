---
date: 2018-11-07T14:50:00+08:00
title: ADS API
description : "介绍Envoy的XDS API中的ADS"
---

 See https://github.com/lyft/envoy-api#apis for a description of the role of
 ADS and how it is intended to be used by a management server. ADS requests
 have the same structure as their singleton xDS counterparts, but can
 multiplex many resource types on a single stream. The type_url in the
  DiscoveryRequest/DiscoveryResponse provides sufficient information to recover
  the multiplexed singleton APIs at the Envoy instance and management server.

有关角色的描述，请参阅https://github.com/lyft/envoy-api#apis
  ADS以及它如何用于管理服务器。 ADS请求
  与他们的单身xDS同行有相同的结构，但可以
  在单个流上复用多种资源类型。 中的type_url
   DiscoveryRequest / DiscoveryResponse提供足够的信息来恢复
   Envoy实例和管理服务器上的多路复用单例API。



```protobuf
service AggregatedDiscoveryService {
  // This is a gRPC-only API.
  rpc StreamAggregatedResources(stream envoy.api.v2.DiscoveryRequest)
      returns (stream envoy.api.v2.DiscoveryResponse) {
  }
}
```

