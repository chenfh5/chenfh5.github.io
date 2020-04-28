---
title: S4-IndicesModule
tags: es
key: 48
modify_date: 2019-04-30 18:00:00 +08:00
---

这个模块涉及检索相关的query builder的注入绑定，

![IndicesModule constructor](https://upload-images.jianshu.io/upload_images/2189341-c076b5be64c14de9.png)

----
# query builder方法
![query方法](https://upload-images.jianshu.io/upload_images/2189341-f739929340a80ed1.png)

其对应着[query-DSL](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-dsl.html)上的方法。

----
# mapping数据类型
![mapping 数据类型](https://upload-images.jianshu.io/upload_images/2189341-96c6628652840c5e.png)

其对应着[mapping datatypes](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/mapping-types.html)上的数据类型。

----
# mapping元数据
![mapping元数据](https://upload-images.jianshu.io/upload_images/2189341-d626303c1f85683a.png)

其对应着[mapping metadata](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/mapping-fields.html)上的数据类型。

----
# 注入绑定项
![IndicesModule所注入的服务](https://upload-images.jianshu.io/upload_images/2189341-01bc379a67c9b6c0.png)

> - bind()中没有to()，这属于无目标绑定，无目标绑定是链接绑定的一种特例，在绑定的过程中不指明目标
> - Eager singletons会尽快启动初始化，保证单例的有效性，Lazy singletons则更适用于edit-compile-run开发周期