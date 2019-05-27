---
title: S3-ClusterModule
tags: es
key: 52
modify_date: 2019-04-30 18:00:00 +08:00
---

这个模块主要将集群的默认配置加上（`registerBuiltinClusterSettings`和`registerBuiltinIndexSettings`），以及绑定一些集群服务（`ClusterInfoService`, `DiscoveryNodeService`, `MetaDataCreateIndexService`, `RoutingService`等）。

![builtin cluster settings](https://upload-images.jianshu.io/upload_images/2189341-cabd27e14b64945b.png)

![builtin index settings](https://upload-images.jianshu.io/upload_images/2189341-0f4783a2b9069940.png)

最后通过interface的configure将以上2个settings的实例（`DynamicSettings.class`）绑定/注入到`ClusterModule.class`上。

![bind setting and service](https://upload-images.jianshu.io/upload_images/2189341-92f3fff5706aeeed.png)

asEagerSingleton作为一种热加载（相较于lazy initialization）。

其中MetaDataCreateIndexService里面就包括了index相关的metadata settings，比如mapping, alias, shards, replicas等。

![create index metadata](https://upload-images.jianshu.io/upload_images/2189341-391cab35463c277c.png)

----
# Guice
在ClusterModule的configure()中，通过bind()将**接口**和**实现类**关联起来。

----
# Reference
- [Google-Guice入门介绍](https://blog.csdn.net/derekjiang/article/details/7231490)
- [Guice系列之用户指南（十）](http://lifestack.cn/archives/143.html)