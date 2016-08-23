# 再论 SolrCloud

经过对 collection 的一番探索，对 SolrCloud 的理解也得到了加深。

## 什么是 SolrCloud

SolrCloud 是一个环境，由多个 node 组成。只要这些 node 使用相同的 zk 配置启动。

有了 SolrCloud 以后，就可以在之上创建 collection