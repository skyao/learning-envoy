---
date: 2018-11-07T15:30:00+08:00
title: Route
weight: 213
menu:
  main:
    parent: "architecture-concept"
description : "概括介绍Envoy中的Route"
---

## Route介绍

路由是一组将虚拟主机(virtual hosts)与群集(cluster)匹配的规则。 

- 虚拟主机 = 域名 + 网址
- virtual hosts = Domain + Path

也就是说：Route的功能是将以 "Domain + Path" 形式展示的虚拟主机映射到集群。

## Route配置

路由通过静态定义或路由发现服务（Route Discovery Service/RDS）进行配置。

### 静态配置

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

### RDS动态配置

路由配置也可以通过 [路由发现服务 (RDS) ](../../xds/rds/) 动态获取。

### 参考资料

- [Routing with a Control Plane](https://www.learnenvoy.io/articles/routing-configuration.html)