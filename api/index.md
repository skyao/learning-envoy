# XDS API

## XDS API简介

XDS API是一系列API的统称，这里的`XDS`代表如下API：

| API简称 | API全称                      | API描述        |
| ------- | ---------------------------- | -------------- |
| EDS     | Endpoint Discovery Service   |                |
| CDS     | Cluster Discovery Service    |                |
| LDS     | Listener Discovery Service   |                |
| RDS     | Route Discovery Service      |                |
| ADS     | Aggregated Discovery Service |                |
| SDS     | Service Discovery Service    | 改名EDS        |
| HDS     | Health Discovery Service     | 设计中，未实现 |
| SDS     | Secret Discovery Service     | 设计中，未实现 |

xds api 在envoy中被成为 `Data plane API`，以下是envoy对这些API的说明：

> 这些API在某些情况下也可以被其他代理解决方案使用，如果这些解决方案也想与管理系统和配置生成器进行互操作，而这些系统和配置生成器是针对此标准构建。因此，我们视这些为通用数据平面API(`universal data plane` API)。 

这里又引出一个 `universal data plane` API 的概念，有时被简称为 `universal API`。

### API定义和维护

xds api 以proto文件的方式定义并被保存在 [Envoy仓库](https://github.com/envoyproxy/envoy) 下的 [api](https://github.com/envoyproxy/envoy/tree/master/api) 子目录下，也可以通过只读仓库 [envoyproxy/data-plane-api](https://github.com/envoyproxy/data-plane-api) 来访问。两者的区别是：

- https://github.com/envoyproxy/envoy/tree/master/api : 可以读写的规范的API保存地点。
- https://github.com/envoyproxy/data-plane-api：上述地址的只读镜像，提供了在没有Envoy实现的情况下使用数据平面API的能力。

目前，xds api 中的 eds/cds/rds/lds 这四个早期定义的API，继续在包 `envoy.api.v2` 中维护，以兼容已有的代码。新的api如 ads 和还在开发中没有实现的 hds/sds 的开发在包  `envoy.service.discovery.v2` 中进行。

[go-control-plane](https://github.com/envoyproxy/go-control-plane) 和 [java-control-plane](https://github.com/envoyproxy/java-control-plane) 两个仓库分别保存着基于Golang和基于Java的API Server 实现代码，实现了在 data-plane-api 中定义的discovery service API。





