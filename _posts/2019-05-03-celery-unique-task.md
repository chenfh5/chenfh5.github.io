---
title: Celery多生产者部署的定时任务去重
tags: python
key: 60
modify_date:
---

# Overview

最近在部署webserver，同一个应用(生产者)部署在3个机器上，3个机器都启用了Celery来启动定时任务，执行文件的增删。

![celery workflow](https://upload-images.jianshu.io/upload_images/2189341-52774c9a5d1d7eda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![application SPF](https://upload-images.jianshu.io/upload_images/2189341-93d9d0df924f7b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到在mq和celery worker阶段都可以做到高可用。但是在user application阶段，存在single point failure的情况，如果单点故障，定时任务就发送不到broker了。

而为了消除user application的SPF或者增加整体吞吐量，一般会部署`多个application`，而这样的情况，每个application都会发送相同的定时任务到broker，导致同一时间就会有多个task。

虽然增删的底层是原子性的，但是多个API同时执行，最后通过conflict来确认是不妥当的。为了解决这个问题，可以采用以下方式，

1. 分布式定时队列
	- 类似oozie，由一个master来控制metadata
	- 将task不写在user application里面，将其再往上抽象一层
2. api发送定时任务到broker之前，先过一层分布式锁，获得锁的application才能发送，
	- mysql字段
	- zookeeper
		- 锁超时：临时节点，node crash之后临时节点自动消除（锁释放）
	- etcd
	- redis(Redission)
3. mq去重，以相同的task id（时间）发送task到mq，然后mq抓取后进行去重（延迟）

其中分布式锁已经有了一个[celery-once](https://github.com/cameronmaske/celery-once)，[celery-singleton](https://github.com/steinitzu/celery-singleton)的实现。

# With Unique Task
![celery worker with unique task](https://upload-images.jianshu.io/upload_images/2189341-81d1a0a97a4cf847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Reference
- [Is it possible to skip delegating a celery task if the params and the task name is already queued in the server?](https://stackoverflow.com/questions/45107418/is-it-possible-to-skip-delegating-a-celery-task-if-the-params-and-the-task-name)
- [Running “unique” tasks with celery](https://stackoverflow.com/questions/4095940/running-unique-tasks-with-celery)
- [任务调度在分布式部署环境下保证task的正确运行](https://my.oschina.net/aiyungui/blog/751882)
- [Celery(四)定时任务](https://www.cnblogs.com/linxiyue/p/8082102.html)
- [再有人问你分布式锁，这篇文章扔给他](https://juejin.im/post/5bbb0d8df265da0abd3533a5)
- [Distributed locks with Redis](https://redis.io/topics/distlock)
