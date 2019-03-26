---
date: 2018-11-07T15:30:00+08:00
title: Listener Filter
weight: 212
menu:
  main:
    parent: "architecture-concept"
description : "概括介绍Envoy中的Listener Filter"
---

### Listener Filter介绍

每个监听器都独立配置有一些网络级别（L3/L4）的过滤器。

当监听器接收到新连接时，配置好的连接本地过滤器将被实例化，并开始处理后续事件。

通用监听器架构用于执行绝大多数不同的代理任务（例如，限速、TLS客户端认证、HTTP 连接管理、 MongoDB sniffing、 原始 TCP 代理等）。

监听器也可以选择性的配置某些监听器过滤器。这些过滤器的处理在网络过滤器之前进行，并有机会操纵连接元数据，通常会影响后续过滤器或集群处理连接的方式。