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