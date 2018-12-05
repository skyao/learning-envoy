---
date: 2018-11-02T10:00:00+08:00
title: Cluster配置参考
weight: 251
menu:
  main:
    parent: "architecture-cluster"
description : "Envoy的Cluster配置参考手册"
---

> 备注：内容来自 https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto

### Cluster配置

配置详细信息实际的源头是来自xDS API 中 Cluster 的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/cds.proto#L50

Cluster配置的JSON格式如下所示：

```json
{
  "name": "...",
  "alt_stat_name": "...",
  "type": "...",
  "eds_cluster_config": "{...}",
  "connect_timeout": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "lb_policy": "...",
  "hosts": [],
  "load_assignment": "{...}",
  "health_checks": [],
  "max_requests_per_connection": "{...}",
  "circuit_breakers": "{...}",
  "tls_context": "{...}",
  "common_http_protocol_options": "{...}",
  "http_protocol_options": "{...}",
  "http2_protocol_options": "{...}",
  "extension_protocol_options": "{...}",
  "dns_refresh_rate": "{...}",
  "dns_lookup_family": "...",
  "dns_resolvers": [],
  "outlier_detection": "{...}",
  "cleanup_interval": "{...}",
  "upstream_bind_config": "{...}",
  "lb_subset_config": "{...}",
  "ring_hash_lb_config": "{...}",
  "original_dst_lb_config": "{...}",
  "least_request_lb_config": "{...}",
  "common_lb_config": "{...}",
  "transport_socket": "{...}",
  "metadata": "{...}",
  "protocol_selection": "...",
  "upstream_connection_options": "{...}",
  "close_connections_on_host_health_failure": "...",
  "drain_connections_on_host_removal": "..."
}
```

具体字段的说明：

| 字段                                     | 格式                             | 说明                                                         |
| ---------------------------------------- | -------------------------------- | ------------------------------------------------------------ |
| name                                     | (string, REQUIRED)               | 提供群集的名称，该群集的名称在所有群集中必须是唯一的。 如果未提供 `alt_stat_name`，则在发出统计信息时使用群集名称。 任何：在发出统计信息时，群集名称中的`:`将转换为`_`。 默认情况下，群集名称的最大长度限制为60个字符。 通过将 `--max-obj-name-len` 命令行参数设置为所需的值，可以增加此限制。 |
| alt_stat_name                            | string                           | 发出统计数据时要使用的群集名称的可选替代项。 任何：在发布统计信息时，名称中的`:`将转换为`_`。。 这不应与路由器过滤器头混淆。 |
| type                                     | Cluster.DiscoveryType            | 用于解析集群的服务发现类型。                                 |
| eds_cluster_config                       | Cluster.EdsClusterConfig         | 用于群集EDS更新的配置。                                      |
| connect_timeout                          | Duration                         | 新建到群集中主机的网络连接的超时。                           |
| per_connection_buffer_limit_bytes        | UInt32Value                      | 集群连接读写缓冲区大小的软件限制。 如果未指定，则使用实现定义的默认值（1MiB）。 |
| lb_policy                                | Cluster.LbPolicy                 | 负载均衡器类型，用于在群集中选择主机。                       |
| ~~hosts~~                                | core.Address                     | 如果服务发现类型是STATIC，STRICT_DNS或LOGICAL_DNS，则需要设置主机。<br/><br/>这个字段已经被废弃，请设置 load_assignment 字段 |
| load_assignment                          | ClusterLoadAssignment            | 设置此选项是指定STATIC，STRICT_DNS或LOGICAL_DNS集群的成员所必需的。 此字段取代hosts字段。 |
| health_checks                            | core.HealthCheck                 | 群集的可选活动运行状况检查配置。 如果未指定任何配置，则不会进行运行状况检查，并且所有集群成员始终被视为运行状况良好。 |
| max_requests_per_connection              | UInt32Value                      | 对单个上游连接的最大请求，可选。 HTTP/1.1和HTTP/2连接池实现都遵循此参数。如果没有指定，则没有限制。将此参数设置为1将有效禁用keep alive。 |
| circuit_breakers                         | cluster.CircuitBreakers          | 集群的熔断，可选。                                           |
| tls_context                              | auth.UpstreamTlsContext          | 用于连接到上游群集的TLS配置。 如果未指定TLS配置，则新连接不会用TLS。 |
| common_http_protocol_options             | core.HttpProtocolOptions         | 处理HTTP请求时的其他选项。 这些选项适用于HTTP1和HTTP2请求。  |
| http_protocol_options                    | core.Http1ProtocolOptions        | 处理HTTP1请求时的其他选项。                                  |
| http2_protocol_options                   | core.Http2ProtocolOptions        | 即使需要默认的HTTP2协议选项，也必须设置此字段，以便Envoy在进行新的HTTP连接池连接时假定上游支持HTTP/2。 目前，Envoy仅支持上游连接的先验知识。 即使TLS与ALPN一起使用，也必须指定http2_protocol_options。 除此之外，这允许HTTP/2连接发生在纯文本上。 |
| extension_protocol_options               | map<string, Struct>              | extension_protocol_options字段用于为上游连接提供特定于扩展的协议选项。 密钥应与扩展过滤器名称匹配，例如“envoy.filters.network.thrift_proxy”。 有关特定选项的详细信息，请参阅扩展的文档。 |
| dns_refresh_rate                         | Duration                         | 如果指定了DNS刷新率且群集类型为STRICT_DNS或LOGICAL_DNS，则此值将用作群集的DNS刷新率。 如果未指定此设置，则默认值为5000毫秒。 对于除STRICT_DNS和LOGICAL_DNS之外的集群类型，将忽略此设置。 |
| dns_lookup_family                        | Cluster.DnsLookupFamily          | DNS IP地址解析策略。 如果未指定此设置，则默认值为AUTO。      |
| dns_resolvers                            | core.Address                     | 如果指定了DNS解析器且集群类型为STRICT_DNS或LOGICAL_DNS，则此值用于指定集群的dns解析器。 如果未指定此设置，则该值默认为默认解析程序，该解析程序使用/etc/resolv.conf进行配置。 对于除STRICT_DNS和LOGICAL_DNS之外的集群类型，将忽略此设置。 |
| outlier_detection                        | cluster.OutlierDetection         | 如果指定，将为此上游群集启用异常值检测。 可以通过运行时值覆盖每个配置值。 |
| cleanup_interval                         | Duration                         | 从群集类型 ORIGINAL_DST 中删除过时主机的时间间隔。 如果在此间隔期间未将主机用作上游目标，则认为主机已过时。 当新连接重定向到Envoy时，新主机会根据需要添加到原始目标群集，从而导致群集中的主机数量随时间增长。 非陈旧的主机（它们被主动用作目的地）保留在群集中，这允许与它们的连接保持打开状态，从而节省了在打开新连接时可能花费的延迟。 如果未指定此设置，则默认值为5000毫秒。 对于ORIGINAL_DST以外的群集类型，将忽略此设置。 |
| upstream_bind_config                     | core.BindConfig                  | 用于绑定新建立的上游连接的可选配置。这将覆盖bootstrap proto中指定的任何bind_config。如果地址和端口为空，则不执行绑定。 |
| lb_subset_config                         | Cluster.LbSubsetConfig           | 负载均衡子集的配置。                                         |
| ring_hash_lb_config                      | Cluster.RingHashLbConfig         | Ring Hash负载均衡策略的可选配置。<br/><br/>LbPolicy选择的负载平衡算法的可选配置。 目前只有RING_HASH和LEAST_REQUEST具有其他配置选项。 指定ring_hash_lb_config或least_request_lb_config而不设置相应的LbPolicy将在运行时生成错误。<br/><br/>只能设置ring_hash_lb_config，original_dst_lb_config，least_request_lb_config中的一个。 |
| original_dst_lb_config                   | Cluster.OriginalDstLbConfig      | 原始目标负载平衡策略的可选配置。<br/><br/>LbPolicy选择的负载平衡算法的可选配置。 目前只有RING_HASH和LEAST_REQUEST具有其他配置选项。 指定ring_hash_lb_config或least_request_lb_config而不设置相应的LbPolicy将在运行时生成错误。<br/><br/>只能设置ring_hash_lb_config，original_dst_lb_config，least_request_lb_config中的一个。 |
| least_request_lb_config                  | Cluster.LeastRequestLbConfig     | LeastRequest负载平衡策略的可选配置。<br/><br/>LbPolicy选择的负载平衡算法的可选配置。 目前只有RING_HASH和LEAST_REQUEST具有其他配置选项。 指定ring_hash_lb_config或least_request_lb_config而不设置相应的LbPolicy将在运行时生成错误。<br/><br/>只能设置ring_hash_lb_config，original_dst_lb_config，least_request_lb_config中的一个。 |
| common_lb_config                         | Cluster.CommonLbConfig           | 所有负载均衡器实现的通用配置。                               |
| transport_socket                         | core.TransportSocket             | 用于上游连接的自定义传输套接字实现，可选。                   |
| metadata                                 | core.Metadata                    | 元数据字段可用于提供有关群集的其他信息。 它可用于统计信息，日志记录和不同的过滤器行为。 字段应使用反向DNS表示法来表示Envoy中哪个实体需要该信息。 例如，如果元数据用于路由器过滤器，则应将过滤器名称指定为envoy.router。 |
| protocol_selection                       | Cluster.ClusterProtocolSelection | 确定Envoy如何选择用于与上游主机通信的协议。                  |
| upstream_connection_options              | UpstreamConnectionOptions        | 上游连接的可选选项。                                         |
| close_connections_on_host_health_failure | bool                             | 如果上游主机变得不健康（由配置的运行状况检查或异常检测确定），立即关闭与故障主机的所有连接。<br/><br/>目前仅支持tcp_proxy创建的连接。<br/><br/>当检测到不健康状态时，此功能的当前实现会立即关闭所有连接。 如果向上游主机开放的大量连接变得不健康，Envoy可能会花费大量时间专门关闭这些连接，而不会处理任何其他流量。 |
| drain_connections_on_host_removal        | bool                             | 如果此群集使用EDS或STRICT_DNS配置其主机，请立即从已从服务发现中删除的任何主机中排除连接。<br/><br/>这仅影响正在进行健康检查的主机的行为。 如果此标志未设置为true，Envoy将等待，直到主机无法进行活动运行状况检查，然后才能将其从群集中删除。 |





