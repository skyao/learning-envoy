---
date: 2018-11-07T14:50:00+08:00
title: RDS
weight: 320
description : "介绍Envoy的XDS API中的RDS"
---

RDS API定义在 `api/envoy/api/v2/rds.proto`:

```protobuf
service RouteDiscoveryService {
  rpc StreamRoutes(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc DeltaRoutes(stream DeltaDiscoveryRequest) returns (stream DeltaDiscoveryResponse) {
  }

  rpc FetchRoutes(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```