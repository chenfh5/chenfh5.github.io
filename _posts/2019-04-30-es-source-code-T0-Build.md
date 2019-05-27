---
title: T0-ES 6.1.4 Gradle build & import to IDEA
tags: es
key: 45
modify_date: 2019-04-30 18:00:00 +08:00
---

# Overview
在es 5.X及之后的版本中，包管理框架从Maven迁移到了Gradle。
- 在Maven导入IDEA的过程中，不需要一些命令行，因为idea的reimport按钮会自动download jar包以及建立索引。
- 而在Gradle中，这个转换过程与mvn有所不同，需要在导入（import project）之前进行一些gradle命令行操作，如下，

1. groovy install
2. gradle install
   - 配置系统环境变量GRADLE_USER_HOME，以便自定义gradle下载的jar包存放位置
3. cd yourDir/es614
4. git clone --depth 1 --branch v6.1.4 https://github.com/elastic/elasticsearch.git
5. cd elasticsearch
6. gradle clean --parallel
7. gradle idea -Dhttp.proxyHost=proxy.your.com -Dhttp.proxyPort=8080 -Dhttps.proxyHost=proxy.your.com -Dhttps.proxyPort=8080 --parallel（不要带http://）
8. gradle build -x test --parallel
9. IDEA import `build.gradle`

![gradle idea begin](https://upload-images.jianshu.io/upload_images/2189341-c3fed7df5c7c75fe.png)

![gradle idea end](https://upload-images.jianshu.io/upload_images/2189341-89ffb63486b16906.png)

![gradle build begin](https://upload-images.jianshu.io/upload_images/2189341-b3314de0f3dc7ef0.png)

gradle build过程中一直加载、编译modules和plugins。

![gradle build mid](https://upload-images.jianshu.io/upload_images/2189341-01bea2e5bbc60c22.png)

上图，在命令行里指定了-x test来跳过测试了，不知道为什么还运行这个main()，这里需要再观察。

![gradle build end](https://upload-images.jianshu.io/upload_images/2189341-d15dd3a0b8402215.png)


上图，虽然最后build failed了，但是将被gradle编译过的es导入到idea之后，还是能够正常显示类关系，即被源码关系链索引好了。

![idea import project](https://upload-images.jianshu.io/upload_images/2189341-83e2dba8af455e35.png)

![import build.gradle](https://upload-images.jianshu.io/upload_images/2189341-c97d5097e514d18c.png)

# Result
![索引后的源码目录](https://upload-images.jianshu.io/upload_images/2189341-f8d70f3dd59624b2.png)

![external libraries第三方库](https://upload-images.jianshu.io/upload_images/2189341-8fd039819d627974.png)

# 遗留问题
1. gradle build -x test的`失效`
2. gradle build的`BUILD FAILED`
3. 为什么没有选择最新的v6.2.4。是由于minimumCompilerVersion的限制。（服务器运行可以是jdk8，但是编译要更新版本的jdk。6.2.x是jdk9；6.3.x是jdk10）

![es tag till 20180508](https://upload-images.jianshu.io/upload_images/2189341-9908147ae79be86a.png)

![BuildPlugin.groovy](https://upload-images.jianshu.io/upload_images/2189341-d551aa0105cdbef8.png)

# Reference
- [Elasticsearch5.5.0源码-编译、导入IDEA、启动](https://www.jianshu.com/p/a22492d40fd1)
- [ElasticStack系列之十六 & ElasticSearch5.x index/create 和 update 源码分析](http://www.cnblogs.com/liang1101/p/7661810.html)
- [gradle命令参数](https://blog.csdn.net/ak471230/article/details/37651381)
- [gradle 命令及技巧 (gradle-tips)](https://juejin.im/entry/58d4c2475c497d0057eaa924)