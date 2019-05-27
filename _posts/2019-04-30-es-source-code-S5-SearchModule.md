---
title: S5-SearchModule
tags: es
key: 50
modify_date: 2019-04-30 18:00:00 +08:00
---

在SearchModule模块中，主要涵盖了`search`性能相关的依赖注入。

![searchModule所挂靠的服务](https://upload-images.jianshu.io/upload_images/2189341-6fcfdd7e5f90ec03.png)

----
# configureSearch
主要是[seach type](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-search-type.html)，search过程一般有两步，首先search带回<_id, _score>，然后fetch带回<_id, _source>。
默认是`query_then_fetch`，shard级别的排序；可以设置为`dfs_query_then_fetch`，cluster级别的全局排序。它们之间的差异可以参见esCn的[柠檬排序](https://link.jianshu.com/?t=https%3A%2F%2Felasticsearch.cn%2Fquestion%2F2275)。

![search方式](https://upload-images.jianshu.io/upload_images/2189341-820700cd1795982b.png)

----
# configureAggs
在此配置[聚合操作](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-aggregations.html)，如SUM，AVG，MIN，MAX，直方图分布等。
![聚合操作](https://upload-images.jianshu.io/upload_images/2189341-6a710cd37b005f4f.png)

在static块上面，进行了es aggregation的三种类型（metrics, bucket, pipeline）注册。

![static的三种aggs注册](https://upload-images.jianshu.io/upload_images/2189341-7f520a4550fef3c9.png)

----
# configureFunctionScore
配置[自定义排序函数](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-dsl-function-score-query.html)，具体函数使用方法可以参考[前文-elasticsearch relevance scoring 检索相关性计算](https://www.jianshu.com/p/8bb84384566a)。

![ScoreFunctionParser的具体实现类](https://upload-images.jianshu.io/upload_images/2189341-a16a6362a4750a10.png)

----
# configureFetchSubPhase
在此注入一些额外信息，比如[explain](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-explain.html), [version](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-version.html), [highlight](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-highlighting.html)等。

![others](https://upload-images.jianshu.io/upload_images/2189341-f171725379213b2b.png)

----
# Reference
- [Java: What is the difference between <init> and <clinit>?](https://stackoverflow.com/questions/8517121/java-what-is-the-difference-between-init-and-clinit)