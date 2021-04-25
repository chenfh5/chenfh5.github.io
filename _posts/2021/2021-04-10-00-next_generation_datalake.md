---
title: datalake关注点
tags: architecture datalake
key: 123
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/114412399-a305a300-9bdf-11eb-8881-2c1ea5c0c81f.png
---

# Overview
记录一下大数据模型的架构演进

# Breakdown
## lambda
目前lambda的大数据类似这样, 一个flink作stream job. 一个spark作batch job来周期性获取stream job的多个output来作model training. 

![datalake](https://user-images.githubusercontent.com/8369671/114396392-199aa480-9bd0-11eb-8a49-190c18b90698.png)
> lambda, 当然这中间可以做出cascade的数据仓库分层模型<sup>4</sup>

![image](https://user-images.githubusercontent.com/8369671/114398735-b65e4180-9bd2-11eb-93d6-7371e101cd74.png)
> lambda arch, credit: ref1
> 
> 上层链路是离线数仓数据流转链路
> 
> 下层链路是实时数仓数据流转链路 

而这个结构里面有不少割裂点,
- compute engine, flink vs spark, 导致2套数据处理逻辑
- storage, stream(kafka) vs batch(s3), 数据没有统一存放, 导致2套schema, 2套数据metadata管理

## kappa

- 如果engine处理得过来, 比如说spark streaming足够low latency, flink batch足够high throughput, 那么compute engine就可以统一起来
- 如果storage处理得过来, 比如low latency write和high throughput read. 支持stream incremental pulling, ACID, flying-CRUD 

![image](https://user-images.githubusercontent.com/8369671/114400225-55376d80-9bd4-11eb-8f27-0b2cfdbe8b70.png)
> kappa arch, credit: ref1
> 
> 上述架构图主要将离线处理链路上的HDFS换成了数据湖iceberg，就号称可以实现实时数仓<sup>1</sup>
> 
> 因此里面最重要的点就是数据湖iceberg. 为什么呢? 是iceberg足够low latency, 足够high throughput吗?

## datalake architecture
数据湖技术目前比较famous的分别采用delta lake, iceberg, hudi<sup>5</sup>

这些`table format`用来协调上层compute engine与下层storage(metadata/format)而引入的中间层(又见架构里面银弹-中间层).

> 这类table format技术是介于上层计算引擎和底层存储格式之间的一个中间层，我们可以把它定义成一种"数据组织格式", iceberg将其称之为"表格式"也是表达类似的含义. 它与底层的存储格式(比如ORC/Parquet之类的列式存储格式, avro等行式存储)最大的区别是它并不定义数据存储方式, 而是定义了数据、元数据的组织方式, 向上提供统一的"表"的语义<sup>6</sup> 

> 它构建在数据存储格式之上, 其底层的数据存储仍然使用Parquet/ORC等进行存储. 当然底层存储介质依旧是hdfs/s3等<sup>6</sup>

### iceberg arch
![image](https://user-images.githubusercontent.com/8369671/114410513-042c7700-9bde-11eb-8ef7-74cb794ac711.png)
> credit: thenewstack

### delta arch
![image](https://user-images.githubusercontent.com/8369671/114410130-b0ba2900-9bdd-11eb-8238-ba6f6cc92a83.png)
> credit: databricks

### hudi arch
![image](https://user-images.githubusercontent.com/8369671/114410718-2b834400-9bde-11eb-8e12-5896efb90959.png)
> credit: xenonstack

### [aws arch](https://aws.amazon.com/cn/solutions/implementations/data-lake-solution/)
![image](https://user-images.githubusercontent.com/8369671/114413431-8f0e7100-9be0-11eb-96b7-cdf676861538.png)
> credit: aws

### [azure arch](https://azure.microsoft.com/en-us/solutions/data-lake/)
![image](https://user-images.githubusercontent.com/8369671/114413411-8ae25380-9be0-11eb-82e6-f94d29446f4e.png)
> credit: azure

# Reference
1. [一文了解实时数据仓库的发展、架构和趋势](https://mp.weixin.qq.com/s/cK6VA7Mnn6F6zdpdiBODPg)
1. [爱奇艺大数据生态的实时化建设](https://mp.weixin.qq.com/s/_7Ht_A2coy2NYiXc5xcC2Q)
1. [图文带你理解Apache Iceberg时间旅行是如何实现的?](https://mp.weixin.qq.com/s/58Si-DIpd3dA2UfGaXvmtQ)
1. [数据建模是什么?](https://cloud.tencent.com/developer/article/1514589)
1. [A Thorough Comparison of Delta Lake, Iceberg and Hudi](https://databricks.com/session_na20/a-thorough-comparison-of-delta-lake-iceberg-and-hudi)
1. [大数据架构变革进行时：为什么腾讯看好 Apache Iceberg?](https://www.infoq.cn/article/59lbbuvcrzlusmdowjbb)
1. [Apache Iceberg快速入门](https://mp.weixin.qq.com/s/LuvN5u9CBPj5AJ_SgJiHsw)
1. [数据工程师眼中的 Delta lake](https://mp.weixin.qq.com/s/Plh1BRp9uZrfIJN5Mi57NA)
1. [基于Flink+Iceberg构建企业级实时数据湖](https://www.bilibili.com/video/BV14A411J7e6?p=4)
1. [aws什么是数据湖?](https://aws.amazon.com/cn/big-data/datalakes-and-analytics/what-is-a-data-lake/)
1. [数据湖(Data Lake)总结](https://zhuanlan.zhihu.com/p/91165577)
