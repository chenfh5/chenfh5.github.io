---
title: S2-DiscoveryModule
tags: es
key: 54
modify_date: 2019-04-30 18:00:00 +08:00
---

在`new Bootstrap().setup()`里实现了instance的初始化，其中主要一步是Node的new，在Node的构造函数当中，将很多module也挂靠上去了。下面简单看看`DiscoveryModule`模块。

![DiscoveryModule constructor](https://upload-images.jianshu.io/upload_images/2189341-03390cc1ecaebff0.png)

DiscoveryModule模块有2类，一个本地版/JVM版`local`，一个集群版`zen`。

# LocalDiscovery
里面有几个重要的field，分别是`ClusterService`，`RoutingService`，`ClusterState`。自成master，`final LocalDiscovery master = firstMaster`。其中`startInitialJoin`用于启动触发，`publish`是master用于下发**集群变化信息**的函数。

![下发信息](https://upload-images.jianshu.io/upload_images/2189341-2dd3ccefff2241e8.png)


主要下发路由信息和元数据信息。

# ZenDiscovery
这个模块相比上一个模块多了集群分布式的功能，如，`TransportService`、`MasterFaultDetection`、`MembershipAction`、`JoinThreadControl`等。
其中，集群模式下的master节点查找功能如下，
`new JoinThreadControl(threadPool)` -> `innerJoinCluster` -> `findMaster()` -> `electMaster()` -> `sortedMasterNodes()` -> `membership.sendJoinRequestBlocking`

![findMaster](https://upload-images.jianshu.io/upload_images/2189341-c555f287970d990d.png)

![master node comparison](https://upload-images.jianshu.io/upload_images/2189341-5ad22758c4961194.png)
