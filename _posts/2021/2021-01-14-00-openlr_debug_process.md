---
title: 自实现openlr的调式过程
tags: java
key: 118
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/104575125-c9c04600-5691-11eb-9a13-c7ae384e8f3c.png
---

# Overview
想通过记录这个调式过程, 方便之后别的项目遇到类似的性能bottleneck问题, 就可以根据这个doc来快速反应.

# background
openlr里面有一个example, unit test里面是mock的openlr.map.[MapDatabase](https://github.com/tomtom-international/openlr/blob/master/maploader-tt-sqlite/src/main/java/openlr/map/sqlite/impl/MapDatabaseImpl.java), 里面是基于`sqlite`来获取数据, 在大规模地图数据的情况下要encode/decode大面积时就非常[耗时](https://github.com/tomtom-international/openlr/issues/33), 所以implement了in-memory模式的[InMemoryMapDatabase](https://github.com/chenfh5/scalatest/blob/master/openlr-way-encoder/src/main/scala/io/github/chenfh5/openlr/lr/MyMapDatabase.scala).

本来以为可以实现快速的编解码, 但是在unit test时却遭遇了滑铁卢, 耗时非常严重.

```shell
[WARN] [01/13/2021 23:22:24.630] [s3mock-akka.actor.default-dispatcher-12] [akka.actor.ActorSystemImpl(s3mock)] Explicitly set HTTP header 'Connection: Keep-Alive' is ignored, illegal RawHeader
[WARN] [01/13/2021 23:22:24.630] [s3mock-akka.actor.default-dispatcher-12] [akka.actor.ActorSystemImpl(s3mock)] Explicitly set HTTP header 'Content-Type: application/octet-stream' is ignored, illegal RawHeader
2021-01-13 23:22:24 INFO  Processor$:95 - getResult=18873501
2021-01-13 23:22:28 INFO  OSMParser:81 - ways size=308539, head=Map(533888964 -> OsmWay([J@2c4087b2,Map(access -> private, highway -> service, oneway -> yes, surface -> asphalt))), last=Map(591394716 -> OsmWay([J@37439861,Map(landuse -> allotments)))
2021-01-13 23:22:28 INFO  OSMParser:82 - nodes size=1611407, head=Map(425355755 -> OsmNode(1.3785712999999948,103.83920129999962))
2021-01-13 23:22:28 INFO  OSMParser:83 - segments size=1797509, head=Map(178569 -> OsmSegment(3191482396,3191482391,313242796))
2021-01-13 23:22:28 INFO  OSMParser:84 - metadata=Metadata(1.2670071000000038,103.68317130000025,1.4690120000000026,103.93744760000006)
2021-01-13 23:22:30 INFO  Processor$:38 - finish fetch wayIDList with time in [5406] ms
2021-01-13 23:58:52 ERROR LocationCheck:235 - location not connected from 784190 to 784191
2021-01-13 23:58:52 ERROR LineLocationCheck:94 - location w is not connected
2021-01-14 00:32:01 INFO  Processor$:68 - finish encoding with time in [4171156] ms
2021-01-14 00:32:01 INFO  Processor$:110 - finish encoding city=SIN with time in [4176914] ms
```

另外还有run了一夜的
```shell
2021-01-14 05:30:29 INFO  Processor$:60 - finish encoding with time in [15902885] ms
2021-01-14 05:30:33 INFO  Processor$:84 - finish encoding city=SIN with time in [15918094] ms

这个跑了4小时, 而面积只是一个Singapore. 所以肯定是InMemoryMapDatabase有bug
```

# find max cpu consumed
之前用过MAT来查OOM的问题, 但是这次比较确认是非OOM问题, 而是cpu的time问题, 所以选择了jprofiler来debug

首先要有一个hprof文件, 
- 使用vm option(-Xmx1g -XX:+HeapDumpOnOutOfMemoryError), 但是这个方式需要crash才生效
- 使用`top + jmap`来生成运行中程序的hprof

在jprofiler里面load进运行中的hprof是snapshot模式, 会出现not available的状况, 此时可以使用`attach`方式来避免snapshot的问题.

![image](https://user-images.githubusercontent.com/8369671/104549415-cf0b9980-566d-11eb-80b5-213dcec60839.png)
> not avaulable while running in snspshot

![image](https://user-images.githubusercontent.com/8369671/104549596-290c5f00-566e-11eb-8c22-76136e84383c.png)
> attach mode

![image](https://user-images.githubusercontent.com/8369671/104552299-83f48500-5673-11eb-9f4b-6b539258aac3.png)
> attach success log

进入到attach mode之后, 选择CPU views - call tree, 可以看到耗时占比, 

![image](https://user-images.githubusercontent.com/8369671/104574670-456dc300-5691-11eb-9130-1bbdbe9e04f8.png)
> 99.2%耗时集中在方块内的方法里

![image](https://user-images.githubusercontent.com/8369671/104575096-c331ce80-5691-11eb-97d4-b09a8c036b63.png)
> 耗时集中在filter

![image](https://user-images.githubusercontent.com/8369671/104575716-4e12c900-5692-11eb-8d39-0d4081fcd6cd.png)
> 定位到问题所在

```scala
val incoming = osmSegmentMap.filter { case (_, s) => s.endNode == nid }.keys.toSet
val outgoing = osmSegmentMap.filter { case (_, s) => s.startNode == nid }.keys.toSet
```

上面代码是在map里面filter, O(n)

## solution 
iterator/filter map -> lookup map, O(n) -> O(1)

定位到bug之后, 就在此优化, 一般都是空间换时间. 在此也是, 在外面生成两个map来存放nodeID所对应的所有segmentID,
```scala
// init
var startnodeSegMap = mutable.Map[Long, mutable.Set[Long]]() // start_id -> seg_id
var endnodeSegMap = mutable.Map[Long, mutable.Set[Long]]() // endnote_id -> seg_id

// lookup
val incoming = endnodeSegMap(nid)
val outgoing = startnodeSegMap(nid)
```
至此, 这个问题fix
```shell
2021-01-14 15:48:40 INFO  Processor$:20 - finish fetch wayIDList with time in [9607] ms
2021-01-14 15:48:59 INFO  Processor$:33 - finish encoding with time in [18862] ms
2021-01-14 15:49:01 INFO  Processor$:57 - finish encoding city=SIN with time in [29883] ms

全跑一个Singapore的encode过程只是29秒.
```

# find largest memory allocated
在load大城市map时, 由于osm很大, 导致in-memory很大. 
获取方式可以跟之前一样是attach, 也可以是那一刻的heap dump snapshot(这里要注意sleep, 因为sleep时object可能已经被释放, 导致失误)

![image](https://user-images.githubusercontent.com/8369671/105000076-db766480-5a68-11eb-9878-70781dec039f.png)
> heap dump snapshot

![image](https://user-images.githubusercontent.com/8369671/105031470-4dae6f80-5a90-11eb-89c3-4a368fb5d265.png)
> 为解决cpu问题而引入的2个map占了60%, 所以要优化

```scala
// V1
val startnodeSegMap = mutable.Map[Long, mutable.Set[Long]]() // start_id -> seg_id
val endnodeSegMap = mutable.Map[Long, mutable.Set[Long]]() // endnote_id -> seg_id
```

将set改为list之后, 占用内存有所下降
```scala
// V2
val startnodeSegMap = mutable.Map[Long, mutable.ListBuffer[Long]]() // start_id -> seg_id
val endnodeSegMap = mutable.Map[Long, mutable.ListBuffer[Long]]() // endnote_id -> seg_id
```

![image](https://user-images.githubusercontent.com/8369671/105031536-64ed5d00-5a90-11eb-9d5d-c86b262fa26f.png)
> V2

## solution
in-memory -> disk storage/off heap

虽然V2使用list替换掉set减轻了负担, 但是in-memory压力还是很大, 所以这里试着引入disk storage/off-heap memory的db, 比如[mapdb](https://github.com/jankotek/mapdb)或者[chronicleMap](https://github.com/OpenHFT/Chronicle-Map), 试了mapdb, 在fileDB SIN的情况下, 跑不成功.

```scala
// @see https://jankotek.gitbooks.io/mapdb/content/performance/
val mapDb = DBMaker
    .fileDB(new File("mdb"))
    .fileMmapEnable() // Always enable mmap
    .fileMmapEnableIfSupported() // Only enable mmap on supported platforms
    .fileMmapPreclearDisable() // Make mmap file faster
    .allocateStartSize(4 * 1024 * 1024 * 1024) // 4GB
    .allocateIncrement(1 * 1024 * 1024 * 1024) // 1GB
    .make()

// @see https://docs.scala-lang.org/overviews/collections-2.13/performance-characteristics.html
val osmWayMap = mapDb.hashMap[Long, OsmWay]("osmWayMap")
val osmNodeMap = mapDb.hashMap[Long, OsmNode]("osmNodeMap")
val osmSegmentMap = mapDb.hashMap[Long, OsmSegment]("osmSegmentMap")
var metadata: Metadata = _

val startnodeSegMap = mapDb.hashMap[Long, mutable.ListBuffer[Long]]("startnodeSegMap")
val endnodeSegMap = mapDb.hashMap[Long, mutable.ListBuffer[Long]]("endnodeSegMap")
```

可能是序列化问题导致. 最后是否可以尝试用rockdb来替代in-memory当storage呢? 但是这样是不是就会退到sqldb呢? 只是在乎的是rockdb的lookup比sql快而已.

最后mapdb还是没有用好, 最后是改了增加了xmx和gc算法才将最大的城市跑成功.
```shell
-Xms21G -Xmx21G -XX:MetaspaceSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=20 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70 -XX:+HeapDumpOnOutOfMemoryError
```

```shell
2021-01-19 23:34:00 INFO  Processor$:47 - city=VN osm data size=163653224
2021-01-19 23:35:39 INFO  OSMParser:91 - ways size=2204151, head=1, last=1
2021-01-19 23:35:39 INFO  OSMParser:92 - nodes size=23928606, head=Map(4276060395 -> OsmNode(15.129832300000077,108.86647209999992))
2021-01-19 23:35:39 INFO  OSMParser:93 - segments size=24614704, head=Map(17938621 -> OsmSegment(4818866840,4818866841,489663435))
2021-01-19 23:35:39 INFO  OSMParser:94 - startnodeSegMap size=23252498, head=Map(1928930788 -> ListBuffer(5907989))
2021-01-19 23:35:39 INFO  OSMParser:95 - endnodeSegMap size=23523299, head=Map(1928930788 -> ListBuffer(5907988))
2021-01-19 23:35:39 INFO  OSMParser:96 - metadata=Metadata(20.1125352,105.96202760000004,20.68090189999999,106.43804389999998)
2021-01-19 23:36:14 INFO  InMemoryEncoder:30 - wayLineMapSort size=2204151, head=Map(550475412 -> List(5795215, 5795216, 5795217, 5795218)), last=Map(853576579 -> List(11850857, 11850858, 11850859, 11850860, 11850861))
2021-01-19 23:36:14 INFO  Processor$:20 - finish fetch wayIDList with time in [133521] ms
2021-01-19 23:56:58 ERROR LocationReferenceAdjust:310 - number of lines are different
2021-01-19 23:56:58 ERROR LocationReferenceAdjust:311 - location: 297 - locRef: 275
2021-01-20 00:09:19 INFO  Processor$:33 - finish encoding with time in [1985557] ms
2021-01-20 00:09:33 INFO  Processor$:61 - finish encoding city=VN with time in [2133169] ms
```

# conclusion
利用这些工具MAT/Jprofiler可以快速定位问题所在. 当然如果一开始就警惕性能问题, 上述的filter从一开始就不会那样写.

但是, 这个定位流程可以适用于之后的debug/optimize.
