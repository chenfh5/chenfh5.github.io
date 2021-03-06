---
title: JM quarantine log
tags: mood
key: 126
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/121048846-bf8e1800-c7e9-11eb-9998-7cf1f875fd38.png
---

# Overview
用日志的方式, 记录这次酒店隔离的点滴

# hotel check-in
汽车从GZ出发, 无缝行驶1h+之后到达JM.

![image](https://user-images.githubusercontent.com/8369671/121017836-4e8e3680-c7d0-11eb-867e-92b0f169d89d.png)
> 酒店落车

接下来是办理入住和咽拭子.

这里说一下现场核酸检测环境对比,

SG是单人间, GZ是连续单人间, JM是露天. 

在JM, 大家扎堆在拥挤的酒店大堂.

![image](https://user-images.githubusercontent.com/8369671/121017857-5352ea80-c7d0-11eb-94a4-eb7b8b665724.png)
> 大堂现场核酸检测

核酸测完, 大家就去等电梯. 

这里就出现很难堪的一幕, 酒店可能是刚刚接受隔离征召, 没有多少经验.

把大堂一分为二(我们这群人在东半边, 西半边不清楚状况), 电梯位于中间serve两边. 隔开之后, 我们在东边按电梯, but only the western side response to the eastern request which should be responded by the eastern server, 而我们过不去西边, 所以就出现dead loop.

等三车人员检测完, 都没人能乘上电梯, haha, terrible management.

![image](https://user-images.githubusercontent.com/8369671/121017863-564ddb00-c7d0-11eb-8b0f-fc8682b50a60.png)
> you will never have chance to take a lift like this

工作做到一半, 把流于表面的空间隔开罢了, 而内在的电梯算法却忽略或者没有意识到. 

最后西边迫不得已开放出来, 大家一窝蜂去了西边(也包括我, 在东边等傻了).

后面的情况就不知道了, 一入房门, 14天内不能踏出.

# room vibe
<div>{%- include extensions/youtube.html id='7aOkh7jN4bw' -%}</div>
> inside

<div>{%- include extensions/youtube.html id='J9OM-nGx3I8' -%}</div>
> outside

# requirement
这边的管理确实有很大进步空间.

比如, 白纸黑字的rule就没有obey.

相比之下, 我觉得钱对他们更重要.

将威胁放在最显眼的位置.

![image](https://user-images.githubusercontent.com/8369671/121034774-df204300-c7df-11eb-8634-06b279b324aa.png)
> money first, threaten second

![image](https://user-images.githubusercontent.com/8369671/121034796-e34c6080-c7df-11eb-8d85-a30cb002eb83.png)
> 1700 dinner???

![image](https://user-images.githubusercontent.com/8369671/121034804-e6475100-c7df-11eb-88a6-9a3a4606d786.png)
> it can not pay in your last quarantine day but the first two days

给管家提了建群意见, 方便消息统一下达. 我们是身体隔离, 而非mind, 一刀切在这里体现得淋漓尽致.

leader给出很大的scope, 下面的明显handle不来, 这就是所谓的揣摩上意, 致使混乱.

如果大家责任与需求明确, rules不朝三暮四. 各司其职, 有序推进.

有秩序就有效率. 

# LOG
## day1
![image](https://user-images.githubusercontent.com/8369671/121037444-09730000-c7e2-11eb-89a8-ca6ff0d7c0f3.png)
> lunch

![image](https://user-images.githubusercontent.com/8369671/121038276-bd748b00-c7e2-11eb-9f10-1b09b786c540.png)
> dinner

![image](https://user-images.githubusercontent.com/8369671/121038292-c1081200-c7e2-11eb-98a8-7d58fcaedc5a.png)
> map

晚上下单了一个mac电池和一些零食(since dinner too early)

## day2
### change mac battery
可能因为personal mac很久没在使用, 加上保养不当.

0601从柜子挖出来再开机, 发现2个问题,
- 电池坏了, 只能走充电线
- 运行变得很卡(even in bare)

卡这个问题很严重. 试着升级了macOS(bigSur), but not work. 

后来怀疑是电池问题造成卡, 但是我自己没有理论和实验支撑这个view.

在网上search和taobao上问客服, 大部分是怀疑battery aging.

但是接线应该就抛开battery工作了, 所以接线应该不会卡, but现实接线后还是卡.

所以在没有conclusion之前, 就想着更换电池了.

因为更换电池至少可以解决接线; 至于卡如果能一并解决就更好.

![image](https://user-images.githubusercontent.com/8369671/121039306-8eaae480-c7e3-11eb-9823-c92b97105d01.png)
> 开盒前

![image](https://user-images.githubusercontent.com/8369671/121039307-8eaae480-c7e3-11eb-92a3-360e4e70d681.png)
> 需要除尘

![image](https://user-images.githubusercontent.com/8369671/121039319-9074a800-c7e3-11eb-9131-049c30699750.png)
> 拆电池

![image](https://user-images.githubusercontent.com/8369671/121039413-a4200e80-c7e3-11eb-9116-b9855a075757.png)
> 螺丝排列

![image](https://user-images.githubusercontent.com/8369671/121054333-d2efb200-c7ee-11eb-93de-54a76e021da3.png)
> 除尘后, 拆卸后, 装新前

![image](https://user-images.githubusercontent.com/8369671/121039454-abdfb300-c7e3-11eb-9008-5c6eba3b1fc5.png)
> 10颗外部螺丝中, top中间2颗是小型

实际上, 换电池后, 系统运行就很流畅了. 

所以, 最后还是battery aging问题导致mac的某些components电压不足吗?

我目前不知道, 需要更多research来support. #TODO

![Xnip2021-06-07_14-20-46](https://user-images.githubusercontent.com/8369671/121039305-8eaae480-c7e3-11eb-8552-e91adfc3c73f.jpg)
> before

![Xnip2021-06-07_15-50-42](https://user-images.githubusercontent.com/8369671/121039304-8eaae480-c7e3-11eb-9135-c27c9e15d9e5.jpg)
> after

![image](https://user-images.githubusercontent.com/8369671/121039463-ae420d00-c7e3-11eb-86ad-ee95660c486a.png)
> recovery :)

----
这边快递速度很快, 昨天下的2单, 今天都到了.
![image](https://user-images.githubusercontent.com/8369671/121043710-750b9c00-c7e7-11eb-9e3d-38f0ce61c2f5.png)
> pantry

![image](https://user-images.githubusercontent.com/8369671/121043833-910f3d80-c7e7-11eb-8dfe-7957cc7ec704.png)
> breakfast

![image](https://user-images.githubusercontent.com/8369671/121836925-0ce91880-cd07-11eb-8216-c62929e9fe27.png)
> lunch

![image](https://user-images.githubusercontent.com/8369671/121836913-08246480-cd07-11eb-99c3-e7a9be11574d.png)
> dinner

![image](https://user-images.githubusercontent.com/8369671/121043834-910f3d80-c7e7-11eb-8f44-c918c44cf1bc.png)
> dining and working corner

![image](https://user-images.githubusercontent.com/8369671/121043836-910f3d80-c7e7-11eb-901f-7085f1c9c379.png)
> mountains, river, and sunset outside the window

![image](https://user-images.githubusercontent.com/8369671/121043837-910f3d80-c7e7-11eb-9c00-b0abe20ec3b6.png)
> fewer steps than before, need more workout 

oops, 啊磊今天recommend的一本书<[囚徒健身](https://book.douban.com/subject/25717097/)>, 或许可以解燃眉之急.

## day3
![image](https://user-images.githubusercontent.com/8369671/121193259-46053100-c8a0-11eb-8111-9b93291f6b66.png)
> breakfast

![image](https://user-images.githubusercontent.com/8369671/121193382-603f0f00-c8a0-11eb-8ba5-a9f8080da040.png)
> lunch

![image](https://user-images.githubusercontent.com/8369671/121836904-035fb080-cd07-11eb-92d4-52127744998d.png)
> dinner

## day4
![image](https://user-images.githubusercontent.com/8369671/121459293-8fa76600-c9dd-11eb-973f-fe14f64b5ff7.png)

![image](https://user-images.githubusercontent.com/8369671/121459310-9635dd80-c9dd-11eb-9b59-45111f8511e8.png)

![image](https://user-images.githubusercontent.com/8369671/121459302-933aed00-c9dd-11eb-83ec-b7285c808d18.png)

![image](https://user-images.githubusercontent.com/8369671/121459315-98983780-c9dd-11eb-80a1-4bb59a29df38.png)

## day5
![image](https://user-images.githubusercontent.com/8369671/121773789-cb465980-cbb0-11eb-9e89-9d427a9afd96.png)

![image](https://user-images.githubusercontent.com/8369671/121773790-ced9e080-cbb0-11eb-9f01-dcf4ab01fc23.png)

![image](https://user-images.githubusercontent.com/8369671/121773794-d26d6780-cbb0-11eb-95e3-e79acf626fef.png)

## day6
![image](https://user-images.githubusercontent.com/8369671/121773959-fa10ff80-cbb1-11eb-8f51-b7ef57c47e5e.png)

![image](https://user-images.githubusercontent.com/8369671/121836882-f478fe00-cd06-11eb-867c-565eada3f9aa.png)

![image](https://user-images.githubusercontent.com/8369671/121773971-07c68500-cbb2-11eb-8637-ca9189316582.png)

## day7
![image](https://user-images.githubusercontent.com/8369671/121773981-1b71eb80-cbb2-11eb-944c-4f3f17236e90.png)

![image](https://user-images.githubusercontent.com/8369671/121773982-1f057280-cbb2-11eb-8287-c76f02d13a4c.png)

![image](https://user-images.githubusercontent.com/8369671/121773987-23ca2680-cbb2-11eb-9126-42e2d6ea45ec.png)

## day8
![image](https://user-images.githubusercontent.com/8369671/121837709-df04d380-cd08-11eb-955e-44cb4fb18af4.png)

![image](https://user-images.githubusercontent.com/8369671/121837711-e2985a80-cd08-11eb-9ecc-a82a374da184.png)

![image](https://user-images.githubusercontent.com/8369671/121837714-e5934b00-cd08-11eb-9f9e-adc94db3b598.png)

![image](https://user-images.githubusercontent.com/8369671/121837717-e88e3b80-cd08-11eb-91a8-28a0f2be79ae.png)

## day9 (Dragon Boat Festival)
![image](https://user-images.githubusercontent.com/8369671/121837929-5b97b200-cd09-11eb-8bfb-e4db5e595d05.png)

![image](https://user-images.githubusercontent.com/8369671/122021295-1ef6b400-cdf8-11eb-920b-ebba4c3fa365.png)

![image](https://user-images.githubusercontent.com/8369671/122021377-33d34780-cdf8-11eb-8519-4f04bf3e1d91.png)

## day10
![image](https://user-images.githubusercontent.com/8369671/122021419-3c2b8280-cdf8-11eb-8f85-2989e854c5a6.png)

![image](https://user-images.githubusercontent.com/8369671/122021442-40f03680-cdf8-11eb-8773-084d7ebddea0.png)

![image](https://user-images.githubusercontent.com/8369671/122418673-f4a22380-cfbc-11eb-8d0d-f701ec37ba0d.png)

## day11
![image](https://user-images.githubusercontent.com/8369671/122418705-fb309b00-cfbc-11eb-8027-fcef6814d024.png)

![image](https://user-images.githubusercontent.com/8369671/122418727-fec42200-cfbc-11eb-9cae-2821ca086f4e.png)

![image](https://user-images.githubusercontent.com/8369671/122418746-02f03f80-cfbd-11eb-8967-72f86b7d732e.png)

## day12
![image](https://user-images.githubusercontent.com/8369671/122418760-071c5d00-cfbd-11eb-9562-bfd238ce0194.png)

![image](https://user-images.githubusercontent.com/8369671/122418779-0aafe400-cfbd-11eb-9370-3ac23a7c61cb.png)

![image](https://user-images.githubusercontent.com/8369671/122418797-0daad480-cfbd-11eb-827e-a0dbaa1b1943.png)

## day13
![image](https://user-images.githubusercontent.com/8369671/122666043-9bfda100-d1dd-11eb-9b81-2c0260bed33e.png)

![image](https://user-images.githubusercontent.com/8369671/122666050-a3bd4580-d1dd-11eb-8a24-57b4d11902fc.png)

![image](https://user-images.githubusercontent.com/8369671/122666046-a029be80-d1dd-11eb-9cbf-5d367f3ae0da.png)

## day14
![image](https://user-images.githubusercontent.com/8369671/122666055-a750cc80-d1dd-11eb-8eea-c0a28ccbb2bb.png)

![image](https://user-images.githubusercontent.com/8369671/122666062-a9b32680-d1dd-11eb-97c4-e659c952a74b.png)

![image](https://user-images.githubusercontent.com/8369671/122666065-ac158080-d1dd-11eb-9cd5-6104c7d0e861.png)

## day15 (Father's Day)
![image](https://user-images.githubusercontent.com/8369671/122666067-af107100-d1dd-11eb-8649-7e95c9d81f6a.png)
> breakfast

![image](https://user-images.githubusercontent.com/8369671/122666068-b172cb00-d1dd-11eb-9b04-450ae4c4d67b.png)
> time counter

![image](https://user-images.githubusercontent.com/8369671/122666071-b46dbb80-d1dd-11eb-9f3d-0c9549c7b4b4.png)

![image](https://user-images.githubusercontent.com/8369671/122666073-b6d01580-d1dd-11eb-9796-92d1dcad8e9e.png)
> 统一下放

![image](https://user-images.githubusercontent.com/8369671/122666075-b9326f80-d1dd-11eb-9feb-b456876e68ff.png)
> pick me up

![image](https://user-images.githubusercontent.com/8369671/122666080-bb94c980-d1dd-11eb-8f0d-24a7631e9f24.png)
> on the way back to the 7 days home isolation
