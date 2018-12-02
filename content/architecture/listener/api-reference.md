---
date: 2018-11-02T10:00:00+08:00
title: Listener配置参考
weight: 221
menu:
  main:
    parent: "architecture-listener"
description : "Envoy的Listener配置参考手册"
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

| 字段             | 格式                           | 说明                                                         |
| ---------------- | ------------------------------ | ------------------------------------------------------------ |
| name             | string                         | 监听器被外界感知的唯一名称。 如果没有提供，Envoy将为监听器分配内部UUID作为名称。 如果要通过LDS动态更新或删除监听器，则必须提供唯一的名称。 默认情况下，监听器名称的最大长度限制为60个字符。 可以通过将 `--max-obj-name-len` 命令行参数设置为所需的值来扩大这个限制。 |
| address          | core.Address, REQUIRED         | 监听器要监听的地址。通常，地址必须是唯一的，尽管它由操作系统的绑定规则控制。 例如，多个监听器可以在Linux上侦听端口0，因为操作系统将分配实际端口。 |
| filter_chains    | listener.FilterChain, REQUIRED | A list of filter chains to consider for this listener. The[FilterChain](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchain) with the most specific [FilterChainMatch](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchainmatch) criteria is used on a connection.为这个监听器准备的过滤器链列表。具有最特定FilterChainMatch标准的FilterChain用于连接。<br/>
使用SNI进行过滤器链选择的示例可以在FAQ条目中找到。 |
| use_original_dst |                                | Example using SNI for filter chain selection can be found in the [FAQ entry](https://www.envoyproxy.io/docs/envoy/latest/faq/sni#faq-how-to-setup-sni). |
|                  |                                |                                                              |
|                  |                                |                                                              |
|                  |                                |                                                              |
|                  |                                |                                                              |
|                  |                                |                                                              |



