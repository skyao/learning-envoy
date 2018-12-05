---
date: 2018-11-07T15:30:00+08:00
title: Endpoint概述
weight: 260
description : "概括介绍Envoy中的Endpoint"
---

## Endpoint介绍

Envoy将“Endpoint”定义为群集中可用的IP和端口。 为了平衡服务中的流量，Envoy希望API提供每个服务的端点列表。 Envoy定期轮询EDS端点，生成响应：

```yaml
version_info: "0"
resources:
- "@type": type.googleapis.com/envoy.api.v2.ClusterLoadAssignment
  cluster_name: some_service
  endpoints:
  - lb_endpoints:
    - endpoint:
       address:
         socket_address:
           address: 127.0.0.2
           port_value: 1234
```

这比定义集群更简单，因为Envoy唯一需要知道的是该端点属于哪个集群。

Envoy 将 CDS/EDS 服务发现视为咨询并最终保持一致; 如果到端点的流量经常失败，则会从负载均衡器中删除端点，直到再次恢复正常。 如果它们不健康，则无需从集群中积极地删除端点。 

## EndPoint配置

### 静态配置

###  EDS动态配置

TBD

###  参考资料

- [Integrating Service Discovery with Envoy](https://www.learnenvoy.io/articles/service-discovery.html)