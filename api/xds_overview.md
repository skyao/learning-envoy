# xDS API概述

> 备注：翻译自 https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md



Envoy通过文件系统或通过查询一个或多个管理服务器来发现其各种动态资源。这些发现服务及其相应的API统称为xDS。 通过订阅，指定要监视的文件系统路径，启动gRPC流或轮询REST-JSON URL来请求资源。后两种方法涉及使用DiscoveryRequest proto 载荷发送请求。在所有方法中资源以DiscoveryResponse proto 负载的形式发送。我们在下面讨论每种类型的订阅。

## 文件系统订阅

提供动态配置的最简单方法是将其放置在 [ConfigSource](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/config_source.proto#core-configsource) 中指定的众所周知的路径中。Envoy将使用`inotify`（Mac OS X上的kqueue）来监视文件的更改，并在更新时解析文件中的`DiscoveryResponse` proto。二进制protobufs，JSON，YAML和proto文本是`DiscoveryResponse`支持的格式。

除了统计计数器和日志以外，没有任何机制可用于文件系统订阅ACK/NACK更新。如果发生配置更新拒绝，xDS API的最后一个有效配置将继续适用。

