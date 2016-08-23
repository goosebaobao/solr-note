# 再探 Collection

如果一个 collection 只有一个 shard，而这个 shard 又只有一个 replica，那和单机模式有神马区别？所以最后一次我来创建一个多 shard 多 replica 的 collection

## 使用默认配置

```
[root@sc78 ~]# /data/solr/bin/solr create -c test2 -shards 2 -replicationFactor 2

Connecting to ZooKeeper at zk:2181/sc ...
Uploading /data/solr/server/solr/configsets/data_driven_schema_configs/conf for config test2 to ZooKeeper at zk:2181/sc

Creating new collection 'test2' using command:
http://localhost:8983/solr/admin/collections?action=CREATE&name=test2&numShards=2&replicationFactor=2&maxShardsPerNode=4&collection.configName=test2

{
  "responseHeader":{
    "status":0,
    "QTime":12983},
  "success":{"172.17.21.78:8983_solr":{
      "responseHeader":{
        "status":0,
        "QTime":3770},
      "core":"test2_shard1_replica2"}}}
```

这次创建的 collection 为 test2，2 个shard，每个 shard 有 2 个 replica，看图

![](sc6.PNG)
很显然，配置依然在 zk 上；那么 core 呢，应该是 4 个吧？

![](sc7.PNG)

查看目录

```bash
[root@sc78 ~]# ll /data/solr/server/solr/
total 36
drwxr-xr-x 5 root root 4096 Jun 21 11:45 configsets
-rw-r--r-- 1 root root 3114 Jun 21 11:45 README.txt
-rw-r--r-- 1 root root 2170 Jun 21 11:45 solr.xml
drwxr-xr-x 3 root root 4096 Aug 23 08:57 test1_shard1_replica1
drwxr-xr-x 3 root root 4096 Aug 23 09:58 test2_shard1_replica1
drwxr-xr-x 3 root root 4096 Aug 23 09:58 test2_shard1_replica2
drwxr-xr-x 3 root root 4096 Aug 23 09:58 test2_shard2_replica1
drwxr-xr-x 3 root root 4096 Aug 23 09:58 test2_shard2_replica2
-rw-r--r-- 1 root root  518 Jun 21 11:45 zoo.cfg
```

## 自定义配置

create 命令可以使用 -d 选项来指定配置目录，如果不指定的话，会使用 data_driven_schema_configs 作为默认配置。test1 和 test2 都是用的默认配置，那么接下来就试下自定义配置吧

### 准备自定义配置



