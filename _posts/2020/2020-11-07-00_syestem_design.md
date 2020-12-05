---
title: interview question on how to design one system
tags: system-design
key: 115
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/99180871-11475600-2765-11eb-8ec3-dcdceb89ce6c.png
---

# Overview
之前看到过关于积分系统的文章<sup>1</sup>, 而这周又看到一个类似的文章<sup>3</sup>. 所以就总结一下关于这4个系统设计面试题,

- 设计一个电商平台积分兑换系统
- 设计一个秒杀系统
- 设计一个微信红包系统
- 设计一个支撑百万用户的IM消息系统

通过对上面的问题的理解, 举一反三, 那么后续的system design应该就比较有把握. 因为上述4个问题大概围绕着缓存, 事务, 中间件, 具体业务场景结合.

# 设计一个电商平台积分兑换系统
数据结构
基石是数据的转移, 数据是使用表来保存, 使用以`表`为设计单元,

```sql
-- 记录了谁拥有多少积分
积分表:
    id (自增id主键)
    user_id (用户id)
    credit (积分)
```

```sql
-- 记录了谁用了多少积分换了什么物品
积分兑换表:
    id (自增id主键)
    user_id (积分表的用户id)
    exchanged_credit (用于兑换的积分)
    product_id (兑换的商品id)
```

```sql
-- 记录了谁用了多少积分换了什么物品
发货申请表:
    id (自增id主键)
    credit_exchange_id (积分兑换表的id)
    type (发货类型，1：购买，2：积分兑换)
    product_id (要发货的商品id)
    product_num (要发货的商品数量)
    express_id (物流单号)
```

技术实现
上面是基本的业务需求和流程梳理, 有了这个业务框架之后, 接下来就是分析其技术实现. 

比如说用户要开始一轮新的兑换, 那么就是三个表同时有动作. 

```sql
积分服务 事务 {
       -> 扣减积分
       -> 新增积分兑换记录
            -> 新增物品发货(这一步可以不同时, 比如说稍晚再发货)
}
```
所以上述就是引入了`MQ`来解耦三个动作,

在MQ中一般有三种语义保证消息的传递. 

- 如果是at least once, 那么就需要考虑重复发货问题
    - 针对漏发问题, 可以通过引入retry来解决
- 如果是at most once, 那么就需要考虑漏发货问题
    - 针对重发问题, 可以通过引入idempotent来解决
- exactly once<sup>2</sup>就是2者综合
    - 可以用`credit_exchange_id`建唯一索引, 保证其有且仅有一条记录(幂等)

![image](https://user-images.githubusercontent.com/8369671/98432773-a494e180-20fc-11eb-9b8f-e90415f22ae3.png)
> from 1

# 设计一个秒杀系统<sup>3,4</sup>
- 超卖问题
- 高并发
    - 服务单一化, 挂这不挂那
    - 单机redis的max-qps约40k, 升级为哨兵; 预加载
    - nginx负载均衡, session/cookie拦截
    - 资源静态化
    - 限流(前端与后端)
    - 异步下单
    - 服务降级
- 接口防刷
    - 黄牛
- 秒杀url<sup>5</sup>
    - 动态生成随机下单页面url, 在下单页面url中追加服务端生成的随机数作为参数
    - 这个参数只能在秒杀活动开始时才能获取。这样即使是网站开发者也无法在秒杀前得到正确的下单url地址
- 数据库设计
    - 单独表格, 隔离性
    - 一张是秒杀订单表，一张是秒杀货品表

![image](https://user-images.githubusercontent.com/8369671/99180804-73ec2200-2764-11eb-909d-706cb2301be4.png)
> from 3

# 设计一个微信红包系统
- 红包类型
    - 个人
    - 群发
- 表结构
    ```
    -- 记录了谁用了多少积分换了什么物品
    红包计数表:
        id
        红包总金额
        剩余红包金额
        红包总个数
        剩余红包个数
    ```
- 业务逻辑
    - 新建/包红包, insert one record
    - 抢红包, 访问cache>0
    - 拆红包 
        - ```sql.excute("update '红包计算表' set remainCount=${total-getAmount}, remainCount=${remainCount-1} where remainCount=${remainCount} and id=${id}")```, remainCount=${remainCount}这里可以避免超卖?

![image](https://user-images.githubusercontent.com/8369671/101234939-de0e3c00-36fe-11eb-8601-609ae2bbf2bf.png)
> set化, 红包id尾部consistent hashing, 各个set之间相互独立，互相解耦。并且同一个红包id的所有请求，垂直流入到同一个set内处理，高度内聚, from 7,8

# 设计一个支撑百万用户的IM消息系统<sup>9</sup>
类似twitter feed, 但是时效性要求更高

群聊类似kafka的consumergroup(一个user一个cgid)

每个会话对应一个Timeline进行消息存储(这里timeline类似list)

core,
- 消息同步, send -> 多端receive, i.e., 多端同步
- 消息存储, 持久化, i.e., 消息漫游 
- 消息检索, search, i.e., 在线检索

表结构
```
-- receiver
接受者表:
    receiver_id
    sender_ids
    endpoint_type(pc, mob, web)
    offset
    time
    
    primary key(receiver_id, sender_id, endpoint_type) // 最后还是需要全端同步
```

```
-- sender
发送者表:
    sender_id
    receiver_ids
    endpoint_type(pc, mob, web)
    msg_list
    offset
    
    primary key(sender_id, receiver_ids, endpoint_type) // sender在不同endpoint登录之后, 根据offset来拉取最新msg_list
```

读放大即pull, 写放大即push问题, 

类似twitter celebrities, 大名会有million的followers, 如果大名(类似trump)发一条消息, 而这条消息在写模式下会被push到所有followers的feed list, 在很多僵尸粉的情况下, 会极为奢侈.

在这样的情况下, 让active 的follower主动去pull这条msg更为合理. 但是在读模式下, 如果该user follow了很多人, 那么user一上线就要去pull million的user.

哪个模式取决该用户的关注数与被关注数. 

- 如果一个用户被很多人关注了, 即其被关注数很多, 那么用pull
- 如果一个用户关注了很多人, 那么用push, 其关注者每发一条msg都放到其待读list


![image](https://user-images.githubusercontent.com/8369671/101237492-34d24080-3714-11eb-90d4-d99beddccdf3.png)
> comparison, from 9

![image](https://user-images.githubusercontent.com/8369671/101237615-3cdeb000-3715-11eb-9ae5-9788b48a3595.png)
> from 9

# Reference
0. [系统设计题：如何设计一个电商平台积分兑换系统](https://mp.weixin.qq.com/s/Egph4Ot4UjqpDrzfADVBDw)
0. [Kafka Exactly Once实现原理](https://cloud.tencent.com/developer/article/1530090)
0. [秒杀架构模型设计](https://mp.weixin.qq.com/s/-nCMiP3t3hqtpB6X4mhCww)
0. [如何设计秒杀系统？](https://www.codenong.com/cs105376522/)
0. [说说网站限时秒杀系统的架构设计](https://www.jianshu.com/p/3ee44857a501)
0. [学会设计一个系统](https://hunzino1.github.io/interview/2019/01/26/project_design.html)
0. [学会设计一个系统](https://yuerblog.cc/wp-content/uploads/2017/10/%E5%BE%AE%E4%BF%A1%E7%BA%A2%E5%8C%85.pdf)
0. [抢红包系统设计](http://maofg.site/arch/2019/01/06/arch-bonus-sys.html)
0. [现代IM系统中的消息系统架构——架构篇](https://www.infoq.cn/article/yPB3Y2lv-DsFtRr5Cguv)
