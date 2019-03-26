---
date: 2018-11-02T10:00:00+08:00
title: Endpoint API
weight: 341
menu:
  main:
    parent: "xds-eds"
description : "Envoy的Endpoint配置参考手册"
---

> 备注：内容来自 https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto

### ClusterLoadAssignment

配置详细信息实际的源头是来自xDS API 中 Endpoint 的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/eds.proto#L43

来自RDS的每个路由将使用 RDS WeightedCluster 中表示的权重映射到单个群集或跨群集的流量分割。

使用EDS，从LB视角看，每个群集都是独立进行处理，LB在群集内的位置之间以及位置内主机之间的更精细粒度之间进行。 对于给定的群集，主机的有效权重是其 `load_balancing_weight` 乘以其 Locality 的 `load_balancing_weight`。

```json
{
  "cluster_name": "...",
  "endpoints": [],
  "policy": "{...}"
}
```

具体字段的说明：

| 字段         | 格式                         | 说明                                                         |
| ------------ | ---------------------------- | ------------------------------------------------------------ |
| cluster_name | (string, REQUIRED)           | 集群的名称。 如果在群集EdsClusterConfig中指定，这将是service_name值。 |
| endpoints    | endpoint.LocalityLbEndpoints | 要负载均衡的端点列表。                                       |
| policy       | ClusterLoadAssignment.Policy | 负载均衡策略设置。                                           |

### ClusterLoadAssignment.Policy

配置详细信息实际的源头是来自xDS API  的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/eds.proto#L54

负载均衡策略设置。

```json
{
  "drop_overloads": [],
  "overprovisioning_factor": "{...}"
}
```

具体字段的说明：

| 字段                    | 格式                                      | 说明                                                         |
| ----------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| drop_overloads          | ClusterLoadAssignment.Policy.DropOverload | 裁剪整体传入流量以保护上游主机的操作。 如果主机无法从中断中恢复，或者由于任何原因无法自动调整或无法处理传入流量，则此操作可以提供保护。<br/><br/>在客户端，每个类别一个接一个地应用，以生成所有传出流量的“实际”丢弃百分比。 |
| overprovisioning_factor | UInt32Value                               | 优先级和地点被认为是过度设置的因素（百分比）。 这意味着我们不认为优先级或地点不健康，直到健康主机的百分比乘以过度配置因子降至100以下。默认值140（1.4），Envoy不认为优先级或地点不健康 直到它们的健康宿主比例降至72％以下。 阅读更多优先级和地区。 |

### ClusterLoadAssignment.Policy.DropOverload

配置详细信息实际的源头是来自xDS API  的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/eds.proto#L57

```json
{
  "category": "...",
  "drop_percentage": "{...}"
}
```

具体字段的说明：

| 字段            | 格式                   | 说明                           |
| --------------- | ---------------------- | ------------------------------ |
| category        | (string, REQUIRED)     | 指定丢弃策略的标识符。         |
| drop_percentage | type.FractionalPercent | 应该为该类别丢弃的流量百分比。 |

