---
title: "Yelp: A stream processing pipeline for an online advertising platform"
tags: spark
key: 6
modify_date: 2019-04-30 18:00:00 +08:00
---

![](http://upload-images.jianshu.io/upload_images/2189341-690b005f45889a08.png)

文章里面提到了2个问题，
1. no state tracking
2. do not support complex customized business logic

![2个待解决问题](http://upload-images.jianshu.io/upload_images/2189341-12475c1d80bd056a.png)

它们通过`updateStateBykey(update_func)/mapWithState(update_func)`来自定义该update过程。即，
1. Attach expire date/time when events are first seen & state is initialized
2. drop the state if it expires
3. apply business logic to new events/current state

![Yelp的解决方案](http://upload-images.jianshu.io/upload_images/2189341-34dde4e6434f53a9.png)

![伪代码](http://upload-images.jianshu.io/upload_images/2189341-79f72927cf4f13b0.png)

我借用了该ppt的思路，试着回答了stackoverflow上面的一个[类似提问](https://stackoverflow.com/questions/44976783/apache-spark-streaming-timeout-long-running-batch/47851670#47851670)，但是我自己没有亲身实现出来，只是觉得可能这是一个思路。在每个state initialization的时候初始化一些状态（timeout，control flag等），然后判断这个stages的去留。
