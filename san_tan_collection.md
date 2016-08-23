# 三探 Coll# 三探 Collection

到现在为止，SolrCloud 仅由一个 node 组成。而我准备了 3 台服务器。很显然，这一次将要在由 3 个 node 组成的 SolrCloud 上创建 Collection

## 启动其他 2 个 node

首先，将需要的 jar 包发送到 sc76，sc77，在 sc78 上执行如下命令

```bash
[root@sc78 ~]# scp /data/solr/dist/ik-analyzer-solr5-5.x.jar root@sc77:/data/solr/dist/.
[root@sc78 ~]# scp /data/solr/dist/mysql-connector-java-5.1.39.jar root@sc77:/data/solr/dist/.

[root@sc78 ~]# scp /data/solr/dist/ik-analyzer-solr5-5.x.jar root@sc76:/data/solr/dist/.
[root@sc78 ~]# scp /data/solr/dist/mysql-connector-java-5.1.39.jar root@sc76:/data/solr/dist/.
```


分别在 sc76，sc77 上以 cloud 模式启动 solr，执行如下命令

```bash
/data/solr/bin/solr start -cloud -z zk:2181/sc
```

如无意外，solr 将成功拉起，进入 solr 管理界面 Cloud - Tree 查看，当前 SolrCloud 已经由 3 个 node 组成，如下图

![](sc9.PNG)

## 创建 collection

