---
date: 2018-11-07T15:30:00+08:00
title: Listener
weight: 211
menu:
  main:
    parent: "architecture-concept"
description : "概括介绍Envoy中的Lisenter"
---

### Listerner介绍

监听器是可以接受来自下游客户端的连接的命名网络位置（如port，unix domain socket等）。

Envoy配置支持在单个进程内有任意数量的监听器。通常我们建议每台机器运行一个Envoy，无论配置的监听器数量如何。这样可以更容易的操作，并有单一的统计来源。 

备注：目前 Envoy 只支持 TCP 监听器。

### Listener配置

监听器配置可以在引导程序配置中静态声明，也可以通过监听器发现服务（Listerner Discovert Service/LDS）动态声明。

顶级 Envoy 配置包含一个监听器列表。每个独立的监听器配置具有以下格式：

```yaml
listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 80
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: backend
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/service/1"
                route:
                  cluster: service1
              - match:
                  prefix: "/service/2"
                route:
                  cluster: service2
          http_filters:
          - name: envoy.router
            config: {}
```

### LDS动态配置

监听器也可以通过 [监听器发现服务 (LDS)](../../xds/lds/)动态获取。



### 参考资料

- [Listeners](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners): 官方文档中的介绍，比较简单
- [Routing Basics](https://www.learnenvoy.io/articles/routing-basics.html): learnenvoy.io 网站的介绍
- [Listerner @ Envoy v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-msg-listener): Envoy v2 API 参考手册中的Listerner一节，这是详细的配置信息。
- [Listener discovery service (LDS)](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/lds): Envoy 配置文档中的LDS一节