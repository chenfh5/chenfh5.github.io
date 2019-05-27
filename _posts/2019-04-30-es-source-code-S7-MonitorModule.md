---
title: S7-MonitorModule
tags: es
key: 41
modify_date: 2019-04-30 18:00:00 +08:00
---

这个模块主要是es集群在系统、进程、JVM级别的性能监控。目录如下，
```
0. Overview
1. 进程相关
2. 系统相关
3. 文件存储相关
4. JVM相关
   - JvmInfo
   - JvmStats
5. Reference
```

----
# Overview
监控频率分为固定一次性fixPoint和refreshInterval、scheduleWithFixedDelay周期性。

![MonitorModule注入的监控服务](https://upload-images.jianshu.io/upload_images/2189341-77825485115c489c.png)

上面所有注入服务可以分为4类，进程相关、系统相关、文件存储相关、JVM相关。

----
# 进程相关
![进程相关](https://upload-images.jianshu.io/upload_images/2189341-285504f85909a8a3.png)

----
# 系统相关
![系统相关](https://upload-images.jianshu.io/upload_images/2189341-ecd02e33f81a63c2.png)

----
# 文件系统相关
![文件系统相关](https://upload-images.jianshu.io/upload_images/2189341-d04fa6a83c058023.png)


----
# JVM相关
jvm监控里面分了2种，一种JvmInfo详情，一种JvmStats统计。

### JvmInfo
![JvmInfo fields](https://upload-images.jianshu.io/upload_images/2189341-706b35d16acf6966.png)

### JvmStats
![JvmStats](https://upload-images.jianshu.io/upload_images/2189341-16e1d729b35bfd5e.png)

如上图JvmStats含有多个重要的fields，如Mem, Threads, GC等

![Mem](https://upload-images.jianshu.io/upload_images/2189341-75206856db2a44e7.png)

![Threads](https://upload-images.jianshu.io/upload_images/2189341-aba5168a3a7acb53.png)

![GC](https://upload-images.jianshu.io/upload_images/2189341-4682f44cab1e5ded.png)

![bufferPool](https://upload-images.jianshu.io/upload_images/2189341-e59265ee6071f59a.png)

![jvm所加载的类统计](https://upload-images.jianshu.io/upload_images/2189341-9359db5f6ba7e560.png)

# Reference
- [Linux中关于swap、虚拟内存和page的区别](https://blog.csdn.net/xifeijian/article/details/8209750)
- [物理内存和虚拟内存](http://uule.iteye.com/blog/2149610)
