---
date: 2018-11-02T10:00:00+08:00
title: Route API
weight: 321
menu:
  main:
    parent: "xds-rds"
description : "Envoy的Route配置参考手册"
---

> 备注：内容来自 https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/rds.proto

### Route配置

配置详细信息实际的源头是来自xDS API 中 Linsenter 的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/rds.proto#L44

Listerner配置的JSON格式如下所示：

```json
{
  "name": "...",
  "virtual_hosts": [],
  "internal_only_headers": [],
  "response_headers_to_add": [],
  "response_headers_to_remove": [],
  "request_headers_to_add": [],
  "request_headers_to_remove": [],
  "validate_clusters": "{...}"
}
```

具体字段的说明：

| 字段                       | 格式                   | 说明                                                         |
| -------------------------- | ---------------------- | ------------------------------------------------------------ |
| name                       | string                 | 路由配置的名称。 例如，它可能与 `config.filter.network.http_connection_manager.v2.Rds`中的 `route_config_name` 匹配。 |
| virtual_hosts              | route.VirtualHost      | 组成路由表的一组虚拟主机(virtual host)。                     |
| internal_only_headers      | string                 | 可选地指定 HTTP header 列表，连接管理器将仅视为内部的。如果在外部请求中找到它们，则会在过滤器调用之前清除它们。 有关更多信息，请参阅 `x-envoy-internal` 。 |
| response_headers_to_add    | core.HeaderValueOption | 指定HTTP header列表，以添加到连接管理器编码的每个响应的。 在此级别指定的 header 将应用于来自任何封闭 `route.VirtualHost` 或 `route.RouteAction` 的header之后。 有关更多信息（包括header值语法的详细信息），请参阅有关自定义请求header的文档。 |
| response_headers_to_remove | string                 | 指定应从连接管理器编码的每个响应中删除的HTTP header 列表。   |
| request_headers_to_add     | core.HeaderValueOption | 指定应添加到HTTP连接管理器路由的每个请求的HTTP header 列表。 在此级别指定的 header 将应用于来自任何封闭`route.VirtualHost` 或 `route.RouteAction` 的标头之后。 有关更多信息（包括 header 值语法的详细信息），请参阅有关自定义请求 header 的文档。 |
| request_headers_to_remove  | string                 | 指定应从HTTP连接管理器路由的每个请求中删除的HTTP header 列表。 |
| validate_clusters          | BoolValue              | 一个可选的布尔值，指定路由表引用的集群是否将由集群管理器验证。如果设置为true且路由引用的群集不存在，则不会加载路由表。如果设置为false并且路由引用的集群不存在，则路由表将加载，如果在运行时选择了路由，路由器过滤器将返回404。如果通过`route_config`选项静态定义路由表，则此设置默认为true。如果通过rds选项动态加载路由表，则此设置默认为false。 在某些情况下，用户可能会覆盖默认行为（例如，将CDS与静态路由表一起使用时）。 |




