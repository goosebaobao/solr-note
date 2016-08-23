# 理解 SolrCloud

经过上面的步骤，实际上我们已经搭建起来 SolrCloud 了。

你也许会说，纳尼？这尼玛就已经把 SolrCloud 搭建起来了？tmd 3 个服务器才用了一个好不好？

稍安勿躁，我们先来理解一下，到底神马是 SolrCloud？

## Solr 单机模式

欲理解 SolrCloud，先理解 Solr 单机模式。有 2 个最基本的概念需要理解

### Solr 实例

所谓 Solr 实例，就是我们运行 bin/solr start 时启动的程序。当一个 Solr 实例启动以后，我们通过 ip 和 端口来访问它。一台服务器上可以运行多个 solr 实例，只要各自的端口不同即可。

默认情况下，我们的 solr 实例使用 8983 端口。

### Solr Core

solr 使用 core 来管理数据。我们都知道，solr 以文档为单位管理数据，所以 core 也就是一个文档的集合。当我们发起一个搜索请求时，实际上是在一个 core 里的所有文档上进行搜索。

一个 solr 实例上可以创建多个 core，每个 core 都是为了满足特定的业务需求而创建的。这有点类似于关系型数据库 mysql，core 就像是 mysql 里的数据库

## SolrCloud

SolrCloud 实际上是一个逻辑概念。我们可以理解为一个或多个 Solr 实例组成了 SolrCloud。相应的，SolrCloud 也有几个基本的概念需要理解

### 逻辑概念

#### collection 集合

collection 和单机模式的 core 是对应的。在 cloud 模式下，服务于某个业务需求的文档集合就是一个 collection。一个 SolrCloud 上可以创建一个或多个 collection。

#### shard 分片

collection 由 shard 组成。或者说，我们把 collection 切分成一个或多个 shard。每个 shard 都拥有 collection 里的部分文档。

### 物理概念

#### node 节点

SolrCloud 由 node 组成。实际上，SolrCloud 里的每个 Solr 实例就是一个 node。一个 SolrCloud 完全可以由单个 node 组成。根据我们对 solr 单机模式的理解，每个 node 上都可以创建一个或多个 core

#### replica 副本

一个 shard 由多个 replica 组成。实际上每个 replica 就是一个 core。也就是说，要创建一个 replica，就必须有一个 core。replica 实际上是由 core 来承载的

如果一个 shard 由多个 replica 组成，那么其中必须有一个是 Leader

一个 collection 的 所有 replica(core) 使用相同的配置

## 理解

所以，什么是 `SolrCloud`？

SolrCloud 可以理解为至少一个 Solr 实例组成的集群。这个集群里可以没有collection，也可以有至少一个 collection，理论上来说，集群里的 collection 是没有上限滴。

那么，什么是 collection？

collection 这个概念离不开 SolrCloud 的环境，单机版的 solr 是木有 collection 的概念滴。

一个 collection 由至少一个 core 组成。所有这些 core 可以在至少一个 solr 实例上。所有这些 core 又划分成了至少一个组，每一组都拥有 collection 的一部分数据，这样的一个组称为一个 shard。每个组里面有至少有一个 core。如果一个组里有一个以上的 core，那么其中必定有一个是 Leader。每一组的多个 core，其数据是一毛一样滴。这些个 core，我们称之为 replica。