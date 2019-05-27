---
title: es Snapshot and Restore
tags: es
key: 23
modify_date: 2019-04-30 18:00:00 +08:00
---

----
# Overview
整理一下es的snapshot功能，分两块，一块是本地磁盘disk存储，一块是远程hdfs作存储，

----
# Version
- elasticsearch-5.4.3.zip
- repository-hdfs-5.4.3.zip

----
# Install plugin
```
# need to specified absolute path
bin/elasticsearch-plugin install file:///data/mapleleaf/es_snapshot/repository-hdfs-5.4.3.zip

# check hdfs master namenode ip and port using webhdfs
curl -i "http://localhost:8081/webhdfs/v1/?op=LISTSTATUS"

# start es
sh bin/elasticsearch -d
ps aux | grep elasticsearch | grep -v "grep" | awk '{print $2}' | xargs kill -9
ps aux | grep elasticsearch | grep -v "grep" | awk '{print $2}' | xargs kill -9 ; sleep 3 && sh bin/elasticsearch -d && ps aux | grep elasticsearch | grep -v "grep" && tailf logs/es_snap.log
```

----
# Disk
## create repo
```
# add below line to esyml
path.repo: ["/data/mapleleaf/es_snapshot/my_backup"]

# create repo, named: my_backup
curl -XPUT 'http://localhost:9200/_snapshot/my_backup' -H 'Content-Type: application/json' -d '{
    "type": "fs",
    "settings": {
        "location": "/data/mapleleaf/es_snapshot/my_backup",
        "compress": true
    }
}'

curl -X GET "localhost:9200/_snapshot/my_backup?pretty"
curl -X DELETE "localhost:9200/_snapshot/my_backup"
```

## create snapshot
```
# create snapshot
curl -X PUT "localhost:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true&pretty"
curl -X GET "localhost:9200/_snapshot/my_backup/*?pretty"
curl -X GET "localhost:9200/_snapshot/my_backup/snapshot_1/_status?pretty"
curl -X DELETE "localhost:9200/_snapshot/my_backup/snapshot_1?pretty"
```

## restore
```
# restore
curl -X POST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore?pretty"
```

----
## setp
1. check index
```
curl -X PUT "localhost:9200/customer" -H 'Content-Type: application/json' -d'
{
    "settings" : {
        "index" : {
            "number_of_shards" : 5,
            "number_of_replicas" : 0
        }
    }
}
'

curl -X GET "localhost:9200/_cat/indices?v"
curl -X DELETE "localhost:9200/customer?pretty"
```

2. insert data
```
for i in {1..10000};
do
    curl -s -X POST "localhost:9200/customer/external/?pretty" -H 'Content-Type: application/json' -d"
    {
      \"id\": ${i},
      \"num\": ${i},
      \"name\": \"John Doe\"
    }" > /dev/null
done
```

![insert docs](https://upload-images.jianshu.io/upload_images/2189341-2113b3fb86ee005c.png)

3. close index
```
curl -X POST "localhost:9200/customer/_close?pretty"
```

4. restore
因为之前我store了一次backup，当时backup只有1条doc，当插入1万条之后，close，然后restore，是以当时store的snapshot来恢复。

![after restore](https://upload-images.jianshu.io/upload_images/2189341-81e3d9c54966069e.png)

5. reinsert
```
curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match_all": {}
    }
}'
```
![reinsert](https://upload-images.jianshu.io/upload_images/2189341-e00899316c9c074a.png)

6. create snapshot_2

![before](https://upload-images.jianshu.io/upload_images/2189341-caa17842c930549a.png)

![after](https://upload-images.jianshu.io/upload_images/2189341-9ff7516ddc60aa54.png)

7 close & restore

----
# [HDFS](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-hdfs-config.html)

## create hdfs repo
```
curl -X PUT "localhost:9200/_snapshot/my_hdfs_repository?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "hdfs",
  "settings": {
    "uri": "hdfs://xxxxx:xxxx",
    "path": "elasticsearch/respositories/my_hdfs_repository",
    "compress": true
  }
}'
```
如果在这一步出现异常，可以参考[这里](https://github.com/elastic/elasticsearch/issues/22156)。

![create repo successed](https://upload-images.jianshu.io/upload_images/2189341-4160fe2a50235645.png)

## insert data
![doc 10000](https://upload-images.jianshu.io/upload_images/2189341-802a2834ab28b162.png)

## create hdfs snapshot
```
curl -X PUT "localhost:9200/_snapshot/my_hdfs_repository/snapshot_hdfs_1?wait_for_completion=true&pretty"
```
![access_control_exception](https://upload-images.jianshu.io/upload_images/2189341-faea66e6dd066368.png)

在`jvm.optiopns`添加插件的安全配置
![fix access_control_exception](https://upload-images.jianshu.io/upload_images/2189341-2867874ad6b456f1.png)

![create snap successed](https://upload-images.jianshu.io/upload_images/2189341-4876eb09b8692f49.png)

![hdfs ls snapshot files](https://upload-images.jianshu.io/upload_images/2189341-98c18b69e64862c4.png)

## restore from hdfs
1. 随意增加一些docs，使得与snapshot时的index有差异，便于观察restore效果。

![doc 10000+](https://upload-images.jianshu.io/upload_images/2189341-a464c2d48ec5bf19.png)

2. close index

![doc index close](https://upload-images.jianshu.io/upload_images/2189341-f24f862cca97bf42.png)

3. restore
curl -X POST "localhost:9200/_snapshot/my_hdfs_repository/snapshot_hdfs_1/_restore?pretty"

![restore successed](https://upload-images.jianshu.io/upload_images/2189341-83c2dd1f32b072e0.png)

![doc 10000](https://upload-images.jianshu.io/upload_images/2189341-33ca2698fdd77b6b.png)

----
# Restoring to a different cluster
> All that is required is `registering` the repository containing the snapshot in the new cluster and `starting` the restore process.
```
curl -X GET "localhost:9201/_cat/indices?v"
```
![clusterB initial](https://upload-images.jianshu.io/upload_images/2189341-46a3906783b1c043.png)

## registering repository
```
curl -X PUT "localhost:9201/_snapshot/my_hdfs_repository?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "hdfs",
  "settings": {
    "uri": "hdfs://xxxxx:xxxx",
    "path": "elasticsearch/respositories/my_hdfs_repository",
    "compress": true
  }
}'
```

![registering using the same hdfs path with clusterA](https://upload-images.jianshu.io/upload_images/2189341-4ff78afb81500689.png)

## list snapshot
```
curl -X GET "localhost:9201/_snapshot/my_hdfs_repository/*?pretty"
```

![lists working snapshots](https://upload-images.jianshu.io/upload_images/2189341-f9744dc4b5535bbe.png)


## starting restore
```
curl -X POST "localhost:9201/_snapshot/my_hdfs_repository/snapshot_hdfs_1/_restore?pretty"
```

![restore successed](https://upload-images.jianshu.io/upload_images/2189341-44e340b0064c7ece.png)

----
# benchmark
会用esrally将数据写入

![before](https://upload-images.jianshu.io/upload_images/2189341-68c24710e32a1174.png)

## snapshoting speed
![hdfs before snapshot](https://upload-images.jianshu.io/upload_images/2189341-a90c827ed4c6681d.png)

```
# backgroud running
curl -X PUT "XXX:9200/_snapshot/my_hdfs_repository/snapshot_hdfs_long_1" -H 'Content-Type: application/json' -d'
{
  "indices": "591_etl_fuhaochen_test_2018062500",
  "ignore_unavailable": true,
  "include_global_state": false
}'

# check running status
curl -X GET "XXX:9200/_snapshot/my_hdfs_repository/*?pretty"
```

![in_progress](https://upload-images.jianshu.io/upload_images/2189341-cc646f4d3e5aa4a0.png)

![success](https://upload-images.jianshu.io/upload_images/2189341-407e3fd4b14e737b.png)

![hdfs after snapshot](https://upload-images.jianshu.io/upload_images/2189341-56f8f1ac2c141c6f.png)

## restoring speed
```
date
curl -X POST "XXX:9201/_snapshot/my_hdfs_repository/snapshot_hdfs_long_1/_restore?wait_for_completion=true&pretty"
date
```

![after](https://upload-images.jianshu.io/upload_images/2189341-88a9afa52d141f07.png)

snapshoting耗时远比restoring高。

----
# plugin auto route
测试一下插件会不会自动路由，即是否需要在每一个节点（datanode，masternode等）都安装？还是只需要在整个es集群的其中一个node安装之后，该node就会将plugin自动路由安装到集群的其他node上？

![health](https://upload-images.jianshu.io/upload_images/2189341-ad4ec44c537f8d88.png)

![nodes](https://upload-images.jianshu.io/upload_images/2189341-3206c93c7c569f0b.png)

![plugins](https://upload-images.jianshu.io/upload_images/2189341-4e20bdfddaba7fba.png)

自动路由不可用。

----
# other
- 尝试snapshot更大的index，但是报错了，配置应该没有问题（因为小索引是snapshot成功的）

![大索引snapshot失败](https://upload-images.jianshu.io/upload_images/2189341-e0774564dbda1d15.png)

![小索引snapshot成功](https://upload-images.jianshu.io/upload_images/2189341-beeab21135570709.png)

[Self-suppression not permitted](https://stackoverflow.com/questions/44490579/what-is-the-main-cause-of-self-suppression-not-permitted-in-spark)这个error应该是hadoop的DataNode剩余空间不够导致。

----
# Reference
- [modules-snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/modules-snapshots.html)
- [repository-hdfs-config](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/repository-hdfs-config.html)
- [SecurityException issue](https://github.com/elastic/elasticsearch/issues/22156)
- [ElasticSearch映射到hdfs的快照](https://my.oschina.net/whx403/blog/911995)
- [ES HDFS快照手册](https://blog.csdn.net/ypc123ypc/article/details/68944108)
- [snapshot探索（增量，incremental）](https://www.jianshu.com/p/59d1cac84e3a)
