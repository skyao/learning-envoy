---
date: 2018-11-07T15:30:00+08:00
title: Route概述
weight: 240
description : "概括介绍Envoy中的Route"
---


## Route介绍

路由(route)是一组将虚拟主机(virtual hosts)与群集(cluster)匹配的规则(rule)，允许您创建流量转移规则。 

Route的功能是将域名 + URL映射到集群。

## Route配置

路由通过静态定义或路由发现服务（Route Discovery Service/RDS）进行配置。

### 静态配置

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

### RDS动态配置

TBD

### 参考资料

- [Routing with a Control Plane](https://www.learnenvoy.io/articles/routing-configuration.html)