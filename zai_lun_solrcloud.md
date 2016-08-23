# 再论 SolrCloud

经过对 collection 的一番探索，对 SolrCloud 的理解也得到了加深。

## 什么是 SolrCloud

SolrCloud 是一个环境，由至少一个 node 组成。任何 node 都可以加入 SolrCloud，只要这些 node 启动时，使用相同的 zk 节点。

有了 SolrCloud 以后，就可以在之上创建 collection