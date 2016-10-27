# python client

python3 可用的库

## SolrClient

[https://github.com/moonlitesolutions/SolrClient](https://github.com/moonlitesolutions/SolrClient)

这个使用比较简单，但是封装的有问题，每个方法都要传 collection，另外好像 commit 有点问题，必须要把 openSearcher 设为 True

## solrcloudpy

[https://github.com/solrcloudpy/solrcloudpy](https://github.com/solrcloudpy/solrcloudpy)

貌似必须要 solr 运行在 cloud 模式，而且在 python3 下有问题，引入了一个 urlparse 模块