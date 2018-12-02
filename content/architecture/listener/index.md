---
date: 2018-11-07T15:30:00+08:00
title: Lister概述
weight: 220
description : "概括介绍Envoy中的Lister"
---

## Listerner介绍

### Listerner是什么？

监听器(Listerner)是可以接受来自下游客户端的连接的命名网络位置（如port，unix domain socket等）。

### 支持哪些Listerner

目前 Envoy 只支持 TCP 监听器。

### Listerner Filter

每个监听器都独立配置有一些网络级别（L3/L4）的过滤器。

当监听器接接收到新连接时，配置好的连接本地过滤器将被实例化，并开始处理后续事件。

通用监听器架构用于执行绝大多数不同的代理任务（例如，限速、TLS客户端认证、HTTP 连接管理、 MongoDB sniffing、 原始 TCP 代理等）。

监听器也可以选择性的配置某些监听器过滤器。这些过滤器的处理在网络过滤器之前进行，并有机会操纵连接元数据，通常会影响后续过滤器或集群处理连接的方式。

## Listener配置

监听器配置可以在引导程序配置中静态声明，也可以通过监听器发现服务（Listerner Discovert Service/LDS）动态声明。

### 静态配置

顶级 Envoy 配置包含一个监听器列表。每个独立的监听器配置具有以下格式：

监听器定义实例如下：

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
			......
          http_filters:
          - name: envoy.router
            config: {}
```

### 动态配置

监听器也可以通过监听器发现服务 (LDS)动态获取。

### 参考资料

- [Listeners](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners): 官方文档中的介绍，比较简单
- [Routing Basics](https://www.learnenvoy.io/articles/routing-basics.html): learnenvoy.io 网站的介绍
- [Listerner @ Envoy v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-msg-listener): Envoy v2 API 参考手册中的Listerner一节，这是详细的配置信息。