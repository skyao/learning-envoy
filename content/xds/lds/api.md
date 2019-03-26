---
date: 2018-11-02T10:00:00+08:00
title: Listener API
weight: 311
menu:
  main:
    parent: "xds-lds"
description : "Envoy的Listener API"
---

> 备注：内容来自 https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-msg-listener

### Listener配置

配置详细信息实际的源头是来自xDS API 中 Linsenter 的 proto 定义文件，地址如下：

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/lds.proto#L38

Listerner配置的JSON格式如下所示：

```json
{
  "name": "...",
  "address": "{...}",
  "filter_chains": [],
  "use_original_dst": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "metadata": "{...}",
  "drain_type": "...",
  "listener_filters": [],
  "transparent": "{...}",
  "freebind": "{...}",
  "socket_options": [],
  "tcp_fast_open_queue_length": "{...}",
  "bugfix_reverse_write_filter_order": "{...}"
}
```

具体字段的说明：

| 字段                              | 格式                           | 说明                                                         |
| --------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| name                              | string                         | 监听器被外界感知的唯一名称。 如果没有提供，Envoy将为监听器分配内部UUID作为名称。 如果要通过LDS动态更新或删除监听器，则必须提供唯一的名称。 默认情况下，监听器名称的最大长度限制为60个字符。 可以通过将 `--max-obj-name-len` 命令行参数设置为所需的值来扩大这个限制。 |
| address                           | core.Address, REQUIRED         | 监听器要监听的地址。通常，地址必须是唯一的，尽管它由操作系统的绑定规则控制。 例如，多个监听器可以在Linux上侦听端口0，因为操作系统将分配实际端口。 |
| filter_chains                     | listener.FilterChain, REQUIRED | 为这个监听器准备的过滤器链列表。最符合 FilterChainMatch 条件的 FilterChain 用于连接。 |
| ~~use_original_dst~~              | BoolValue                      | 如果使用 iptables 重定向连接，则代理接收到的端口可能与原始目标地址不同。当此标志设置为 true 时，监听器将重定向的连接切换到与原始目标地址关联的监听器。 如果没有与原始目标地址关联的监听器，则连接由接收它的监听器处理。 默认为false。<br/><br/> 这个字段已经被废弃。取而代之的是 original_dst listener filter |
| per_connection_buffer_limit_bytes | UInt32Value                    | 对监听器新建连接的读写缓冲区大小的软限制。如果未指定，则使用实现定义的默认值（1MiB）。 |
| metadata                          | core.Metadata                  | 监听器元数据                                                 |
| drain_type                        | Listener.DrainType             | 在监听器范围内执行的逐出类型。                               |
| listener_filters                  | listener.ListenerFilter        | 监听器过滤器有机会操纵和扩充连接过滤器链匹配中使用的连接元数据。 这些过滤器在 filter_chains 中的过滤器之前运行。 处理过滤器的顺序很重要：在监听器接受套接字之后，并在创建连接之前。 |
| transparent                       | BoolValue                      | 是否应将监听器设置为透明套接字。<br/><br/>当此标志设置为true时，可以将连接重定向到使用iptables TPROXY 目标的监听器，在这种情况下，original source 和 destination address以及端口将保留在已接受的连接上。此标志应与original_dst 监听器过滤器结合使用，以将连接的本地地址标记为“restored”。这可用于将每个重定向连接切换到与该连接的目标地址关联的另一个监听器。不使用TPROXY而直接连到套接字的连接无法与使用TPROXY重定向的连接区分开来，因此它们被视为重定向。当此标志设置为false时，监听器的套接字将显式重置为非透明。设置此标志需要Envoy以CAP_NET_ADMIN功能运行。如果未设置此标志（默认），则不修改套接字，即既不设置也不重置透明选项。 |
| freebind                          | BoolValue                      | 监听器是否应设置 IP_FREEBIND 套接字选项。 当此标志设置为true时，可以将监听器绑定到未在运行Envoy的系统上配置的IP地址。 当此标志设置为false时，套接字上的选项 IP_FREEBIND 被禁用。 如果未设置此标志（默认值），则不会修改套接字，即既未启用也未禁用该选项。 |
| socket_options                    | core.SocketOption              | Envoy源代码或预编译二进制文件中可能不存在的其他套接字选项。  |
| tcp_fast_open_queue_length        | UInt32Value                    | 监听器是否应接受 TCP Fast Open（TFO）连接。 当此标志设置为大于0的值时，将在套接字上启用选项TCP_FASTOPEN，其队列长度为指定大小（请参阅RFC7413中的详细信息）。 当此标志设置为0时，套接字上禁用选项TCP_FASTOPEN。 如果未设置此标志（默认值），则不会修改套接字，即既未启用也未禁用该选项。 |



### 枚举类型 Listener.DrainType

- DEFAULT

	*(默认值)* ⁣逐出发生在调用`/healthcheck/fail`管理端点（以及运行健康检查过滤器）的应答，监听器移除/修改和热重启。

- MODIFY_ONLY

	⁣逐出发生监听器移除/修改和热重启。 此设置不包括 `/healthcheck/fail`。如果Envoy同时托管ingress和egress监听器，则可能需要此设置。


