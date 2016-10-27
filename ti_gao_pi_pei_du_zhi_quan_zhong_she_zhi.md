# 提高匹配度之权重设置

## 参数介绍

DisMax/eDisMax 有如下 2 个参数

### qf

查询字段，在执行查询时用哪些字段来匹配搜索条件，如果为设置，则使用 `df` 参数指定的字段

该字段必须创建了索引，否则查不到数据

## pf

短语匹配加权的字段：这些字段如果做到短语匹配会加权

首先要明白什么是短语：```hello world``` 分词以后得到 2 个词条：```hello``` 和 ```world```，但是如果我想要查询的是整个 ```hello world``` 呢？可以用引号 ```"hello world"``` 来表示这是个短语，就不会被分词了

如果字段里匹配到了整个短语(不包括匹配到了构成短语的所有单词但并非一个短语)，那么这个字段相比其他字段权重会更大

例如，有如下 2 个文档

```
1. hello world solr
2. hello solr world
```

如果查询条件为 ```hello world``` 那么文档 1 的权重会大于文档 2，因为文档 1 做到了短语匹配

boosts the score of documents in cases where all of the terms in the q parameter
appear in close proximity.