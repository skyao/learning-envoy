---
date: 2018-11-07T15:30:00+08:00
title: 概念与术语
menu:
  main:
    parent: "architecture"
weight: 210
description : "介绍Envoy的概念与术语"
---

Envoy中的概念与术语

## Listener

监听器(Listerner)是可以接受来自下游客户端的连接的命名网络位置（例如，port，unix doain socket等）。 Envoy暴露了一个或多个监听器。

监听器配置可以在引导程序配置中静态声明，也可以通过监听器发现服务（Listerner Discovert Service/LDS）动态声明。

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

## 

## Route

路由(route)是一组将虚拟主机(virtual hosts)与群集(cluster)匹配的规则(rule)，允许您创建流量转移规则。 

路由通过静态定义或路由发现服务（Route Discovery Service/RDS）进行配置。

### 定义Route

Envoy的路由(route)定义将域名(domain) + URL映射到集群(cluster)。

下面的例子中的virtual_hosts定义了两个路由规则，匹配到两个集群(Cluster)：

```yaml
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
```

请记住，RDS规范只是传输机制

## Cluster

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



## Endpoint

Envoy将“端点(Endpoint)”定义为群集(Cluster)中可用的IP和端口。 为了平衡服务中的流量，Envoy希望API提供每个服务的端点列表。



参考：

https://www.learnenvoy.io/articles/routing-configuration.html