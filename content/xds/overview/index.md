---
date: 2018-11-07T14:50:00+08:00
title: xDS概述
weight: 302
description : "介绍Envoy的XDS API"
---

## xds通用模式

LDS/EDS/CDS/EDS 这四个xDS API的定义非常类似，模式都是一样的。

LDS：

```protobuf
service ListenerDiscoveryService {
  rpc StreamListeners(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchListeners(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

RDS：

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

CDS：

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

EDS：

```protobuf
service EndpointDiscoveryService {
  rpc StreamEndpoints(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchEndpoints(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

模式都是通用的：

- 都有一个单次调用的 `Fetch***` 方法和一个gRPC双向流的  Stream***` 
- 而且四个xDS API的这两个方法的参数都是一样的：DiscoveryRequest / DiscoveryResponse
- RDS和CDS 还有用于增量更新的 `Delta***`，参数也相同：DeltaDiscoveryRequest / DeltaDiscoveryResponse

## ADS

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

模式其实和LDS/EDS/CDS/EDS 这四个xDS API的定义是一样的。



