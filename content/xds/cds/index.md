---
date: 2018-11-07T14:50:00+08:00
title: CDS
weight: 330
description : "介绍Envoy的XDS API中的CDS"
---

CDS API定义在 `api/envoy/api/v2/cds.proto`:

```protobuf
service ClusterDiscoveryService {
  rpc StreamClusters(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc DeltaClusters(stream DeltaDiscoveryRequest) returns (stream DeltaDiscoveryResponse) {
  }

  rpc FetchClusters(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```