---
date: 2018-11-07T15:30:00+08:00
title: Cluster概述
weight: 250
description : "概括介绍Envoy中的Cluster"
---

## 

群集(cluster)是一组类似的上游主机(hosts)，接受来自Envoy的流量。集群允许均衡服务集的负载平衡，以及更好的基础架构弹性。 

In Envoy’s vernacular, a “cluster” is a named group of hosts/ports, over which it will load balance traffic. 

在Envoy的白话中，“群集(cluster)”是一组命名的主机(host)/端口(port)，Envoy将通过群集来实现流量的负载均衡。

通过静态定义或使用群集发现服务（Cluster Discovery Service/CDS）配置群集。

集群定义实例如下：

```yaml
clusters:
  - name: service1
      connect_timeout: 0.25s
      type: strict_dns
      lb_policy: round_robin
      http2_protocol_options: {}
      hosts:
      - socket_address:
          address: service1
          port_value: 80
  - name: service2
      connect_timeout: 0.25s
      type: strict_dns
      lb_policy: round_robin
      http2_protocol_options: {}
      hosts:
      - socket_address:
          address: service2
          port_value: 80
```

