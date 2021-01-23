---
title: join implement and classification in big data processing flamework
tags: architect flink spark
key: 119
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/105574258-3d530900-5d9e-11eb-96bf-28e0edf14ee6.png
---

# Overview
整理一下自己关于flink/spark join的理解

# Join [algorithms](https://en.wikipedia.org/wiki/Category:Join_algorithms)
## nested loop join, O(n^2)
```python
for each tuple r in R do
    for each tuple s in S do
        if r and s satisfy the join condition then
            yield tuple <r,s>
```
![image](https://user-images.githubusercontent.com/8369671/105963430-f45fc500-60bb-11eb-80b1-d26ee9f0b423.png)
> credit: csdn

## index nested loop join, O(nlogn)
```python
for each tuple r in R do
    for each tuple s in S in the index lookup do
        yield tuple <r,s>
```

![image](https://user-images.githubusercontent.com/8369671/105963455-fa55a600-60bb-11eb-9d41-e677737ee002.png)
> reduce the lookup time in S, since index, credit: csdn
    
## block nested loop, O(nlogn)
```scala
for each tuple r in R do
    add r to in_memory_hashtable
    if size(hashtable) > max_memory_size
        scan_S_to_join
        in_memory_hash_table.clear()
final scan of S with rest hashtable

// assume size R < S
// not every record of R scan S, but load R as a batch(block/map/page),
// then iterator S to lookup R.

// A special-case of the classic hash join.
```
![image](https://user-images.githubusercontent.com/8369671/105963930-88ca2780-60bc-11eb-9cca-52d9fea6eb25.png)
> reduce the loop time in S, since buffer, credit: csdn

## hash join
```scala
add r to in_memory_hashtable // if memory not sufficiency then degrade to block nested loop
for each tuple s in S do
    if r and s satisfy the join condition then
        yield tuple <r,s>
```
![image](https://user-images.githubusercontent.com/8369671/105971253-49540900-60c5-11eb-8426-004ca10c9c8b.png)
> credit: hive

### grace hash join
```scala
// raw hash join + partition join key(disk),
// let partition data can fit memory, 
// and since partiton, no shuffle required

for row in t1:
    hashValue = hash_func(row)
    N = hashValue % PartCount;
    write row to file t1_N;

for row in t2:
    hashValue = hash_func(row)
    N = hashValue % PartCount;
    write row to file t2_N;

for i in range(0, PartCount):
    hash_join(t1_i, t2_i)
```
![image](https://user-images.githubusercontent.com/8369671/105971449-7f918880-60c5-11eb-9181-0e093ca21cef.png)
> add partition, credit: hive

### hybird hash join
```scala
partition + try keeping first partition in memory to avoid one more disk load67
```
![image](https://user-images.githubusercontent.com/8369671/105983906-5debcd80-60d4-11eb-99c2-ee8c883dbd69.png)
> partition0 hot start, credit: hive

### symmetric hash join
```scala
design for streaming
```
![image](https://user-images.githubusercontent.com/8369671/105984637-6264b600-60d5-11eb-974c-936a1b05116a.png)
> credit: jdon

## sort merge join
```python
// S is bigger
// 1. sort R, S on join keys
// 2. merge

r <- sortedR, s <- sortedS
while r and s:
  if r > s: increment s
  if r < s: increment r
  elif r match s on join key: 
      yield tuple <r,s>
      increment s
  else None
```
![image](https://user-images.githubusercontent.com/8369671/106003201-eecea300-60ec-11eb-8db3-272ea2ac079b.png)
> sort, iterator not from very beginning merge, credit: cnblogs


# Classification
## flink
### [table join](https://github.com/apache/flink/blob/release-1.12.1/flink-table/flink-table-api-java/src/main/java/org/apache/flink/table/api/internal/TableImpl.java#L225)
0. base on [calcite](https://calcite.apache.org/)
0. generator
    - TABLE_EXEC_DISABLED_OPERATORS
0. planner
    - OperatorType.SortMergeJoin, JoinType.INNER
    - PlannerBase -> ExecNodeGraphGenerator -> translateToPlan
    - translateToPlanInternal -> HashJoinOperator/SortMergeJoinOperator
0. runner
    - executor JoinOperator 

### [dataset join](https://github.com/apache/flink/blob/release-1.12.1/flink-scala/src/main/scala/org/apache/flink/api/scala/DataSet.scala#L1129)
0. set joinHint(BROADCAST_HASH, REPARTITION_SORT_MERGE)
0. sdet joinType(INNER, OUTER)
0. compileJobGraph().preVisit() -> JoinNode()
0. switch (joinHint) -> choose broadcast-hash or merge-sort
0. richFlatJoinFunc()

### [stream join](https://github.com/apache/flink/blob/release-1.12.1/flink-streaming-scala/src/main/scala/org/apache/flink/streaming/api/scala/DataStream.scala#L922)
- unbounded
    - `symmetric hash join`
- bounded/window/interval
    0. union two stream, making it like one stream
        - `unionStream = taggedInput1.union(taggedInput2)`
    0. apply this unioned stream with richFlatJoinFunc()

## spark
### [table join](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html)
### [dataset join](https://spark.apache.org/docs/latest/rdd-programming-guide.html#JoinLink)
### [stream join](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#join-operations)
- unbounded
    0. buffer past input as streaming state
    0. limit the state using watermarks
- bounded/window/interval

----
> ps. 大数据处理framework有一个好处就是shuffle后天然是sort的, 有利于merge-sort join, 这也是为什么它们在大数据方面很优异. 当然核心思想还是分治, 分治后一个partition的join就相当于跑在普通机器的sql-lite.

# Realtime Query on Stream
如何在一个stream中实时作query,
- flink是[dynamic table](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/streaming/dynamic_tables.html)
    ![image](https://user-images.githubusercontent.com/8369671/106353321-9d105d80-6324-11eb-996b-16e384906343.png)
    > stream -> dynamic table(DT) -> query on DT -> new DT -> stream, credit: flink
    
    ![image](https://user-images.githubusercontent.com/8369671/106353481-d4cbd500-6325-11eb-8ba4-e101a3bef15d.png)
    > Liz is the latest record, i.e, (Mary, /home) comes first, credit: flink
- spark是[unbound table](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#basic-concepts)
    ![image](https://user-images.githubusercontent.com/8369671/106352992-2d00d800-6322-11eb-8b64-197ebe09478a.png)
    > new arrival stream append to a table, each query trigger equal to execute SQL on this table, credit: spark

    ![image](https://user-images.githubusercontent.com/8369671/106352528-e9589f00-631e-11eb-9b3a-6538f329cbe7.png)
    > credit: spark

# Reference
0. [Peeking into Apache Flink's Engine Room, dataset only](https://flink.apache.org/news/2015/03/13/peeking-into-Apache-Flinks-Engine-Room.html)
0. [Flink使用Broadcast State实现流处理配置实时更新](http://shiyanjun.cn/archives/1857.html)
0. [The Broadcast State Pattern](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/broadcast_state.html)
0. [Joining](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/operators/joining.html)
0. [FLINK入门——JOIN整理](https://juejin.cn/post/6844903922771951630)
0. [Broadcast “JOIN” in Flink](https://stackoverflow.com/a/58426935)
0. [学习Mysql的join算法：Index Nested-Loop Join和Block Nested-Loop Join](https://blog.csdn.net/u010841296/article/details/89790399)
0. [cmu join algorithms](https://zhenghe.gitbook.io/open-courses/cmu-15-445-645-database-systems/join-algorithms)
0. [Spark Streaming vs. Structured Streaming](https://dzone.com/articles/spark-streaming-vs-structured-streaming)
0. [Structured Streaming VS Flink in zhihu](https://zhuanlan.zhihu.com/p/54418431)
