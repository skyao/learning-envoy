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

### LDS动态配置

监听器也可以通过监听器发现服务 (LDS)动态获取。

监听器发现服务（LDS）是一个可选的 API，Envoy 将调用它来动态获取监听器。Envoy 将协调 API 响应，并根据需要添加、修改或删除已知的监听器。

监听器更新的语义如下：

- 每个监听器必须有一个唯一的名字。如果没有提供名称，Envoy 将创建一个 UUID。要动态更新的监听器，管理服务必须提供监听器的唯一名称。
- 当一个监听器被添加，在参与连接处理之前，会先进入“预热”阶段。例如，如果监听器引用 RDS 配置，那么在监听器迁移到 “active” 之前，将会解析并提取该配置。
- 监听器一旦创建，实际上就会保持不变。因此，更新监听器时，会创建一个全新的监听器（使用相同的侦听套接字）。新增加的监听者都会通过上面所描述的相同“预热”过程。
- 当更新或删除监听器时，旧的监听器将被置于 “draining（逐出）” 状态，就像整个服务重新启动时一样。监听器移除之后，该监听器所拥有的连接，经过一段时间优雅地关闭（如果可能的话）剩余的连接。逐出时间通过 [`--drain-time-s`](http://www.servicemesher.com/envoy/operations/cli.html#cmdoption-drain-time-s) 选项设置。

> **注意**： 任何在 Envoy 配置中静态定义的监听器都不能通过 LDS API 进行修改或删除。

### 参考资料

- [Listeners](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners): 官方文档中的介绍，比较简单
- [Routing Basics](https://www.learnenvoy.io/articles/routing-basics.html): learnenvoy.io 网站的介绍
- [Listerner @ Envoy v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-msg-listener): Envoy v2 API 参考手册中的Listerner一节，这是详细的配置信息。
- [Listener discovery service (LDS)](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/lds): Envoy 配置文档中的LDS一节