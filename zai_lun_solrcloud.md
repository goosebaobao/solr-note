# 再论 SolrCloud

经过对 collection 的一番探索，对 SolrCloud 的理解也得到了加深。

## 什么是 SolrCloud

SolrCloud 是一个环境，由至少一个 node 组成。任何 node 都可以加入 SolrCloud，只要该 node 启动时，使用相同的 zk 节点。

当然，也有例外，如果一个 node 在加入 SolrCloud 以后，发现自己的某个 core 原本不属于该 SolrCloud，那么这个 node 应该会爆出异常。什么意思呢？假定一个 node 原本属于 CloudA，但在一次启动时传递了错误的 -z 参数而加入了 CloudB，该 node 想要启动自己的 core，却无法在 zk 上找到配置，这时至少这些 core 肯定是无法启动的。

也就是说，一个新的 node 要加入一个 SolrCloud，那么它本身应该是没有任何 core 的。

那么，如何区分 2 个不同的 SolrCloud？只有通过其 zk 的节点来区分，也就是说，实际上是 zk 上的某个节点唯一标识了 SolrCloud。

有了 SolrCloud 以后，就可以在之上创建 collection，从这个意义上来说，SolrCloud 是个环境，用于创建和承载 collection 的环境

## 什么是 node

一个 node 就是一个 solr 进程，或曰 solr 实例。它有 2 个关键属性：ip 和 端口。

这也意味着，多个 node 可以共存在同一个物理服务器上。所以，不能仅根据 node 的数量是否大于 1 来判断 SolrCloud 是否高可用。

## 什么是 collection

理解 collection，可以把它想象成数据库。Solr 是一个搜索引擎，其核心功能是对海量的 document 进行索引，并在用户发起搜索请求时，找到匹配的 document 返回给用户。

那么 collection 就是一组 document 的集合，无论是索引，查询，都是相对一个 collection 来说的。针对不同的业务，可以创建不同的 collection。

一个 SolrCloud 环境可以创建多个 collection，各个 collection 之间是互相独立的。

## 什么是 shard

一般来说，搜索引擎要处理的都是海量的数据。对于海量数据的处理，一个流行的做法就是分片。

把一个 collection 划分成多份，每一份就是一个 shard。显然，不同的 shard 拥有不同的 document，一个 collection 能对外提供服务的前提是所有 shard 都正常。

将一个 collection 切分为多少个 shard 取决于

1. collection 里的 document 数量，并不仅仅是实际数量，还应考虑到未来的增量，即理论上的总量
2. 搜索请求的并发数

显然，规划 shard 数量，既要考虑到更易于添加、索引、存储海量的文档，也要考虑到搜索的并发性能。

## 什么是 replica

collection 划分为多个 shard 后，实际上增加了可用性的风险：任意一个 shard 不可用，都会导致整个 collection 不可用。为了提高可用性，shard 应该有冗余，这就有了 replica 的概念

每个 replica 都拥有 shard 的所有数据。只有所有 replica 都不可用，shard 才会不可用。

当一个 shard 有多个 replica 时，其中必须有一个是 Leader replica。这是因为 replica 不仅是数据的冗余，还负责对用户的搜索请求进行处理。

所以说，replica 实际是 shard 的物理具现。从物理上来说，每个 replica 都以 core 的形式存在。同一个 collection 里的所有 replica，使用同样的配置。

那么，如何决定一个 shard 有几个 replica 呢？

1. 取决于可用性要求。replica 越多，可用性越高。当然，前提是 replica 在不同的服务器上。如果有 3 台服务器，那么每个 shard 有 3 个 replica 的可用性肯定大于 2 个 replica。
2. 取决于性能要求，即搜索请求的并发数。并发数越大，要求的 replica 越多。这是因为 replica 是以 core 的形式存在的，而搜索请求最终总是由 core 来执行的。

建议 replica 的数量不要高于物理服务器的数量，因为 replica 的数量等于物理服务器的数量时已经达到最大的可用性了，再多的话。。。哦，考虑到磁盘的故障，例如某个扇区的损坏，貌似更多的 replica 对于保证数据的安全还是有意义滴
