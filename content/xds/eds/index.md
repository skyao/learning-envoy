---
date: 2018-11-07T14:50:00+08:00
title: EDS
weight: 340
description : "介绍Envoy的XDS API中的EDS"
---

EDS API定义在 `api/envoy/api/v2/eds.proto`:

```protobuf
service EndpointDiscoveryService {
  rpc StreamEndpoints(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchEndpoints(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```