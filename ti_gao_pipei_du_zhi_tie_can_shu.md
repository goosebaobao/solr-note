# 提高匹配度之 tie 参数

在引入 tie 参数之前，看一个例子

有如下的文档

```python
    albums = (
        Album(name='钢铁战士', desc='蜘蛛侠'),
        Album(name='钢铁意志', desc='猪猪侠奇遇记'),
        Album(name='国家宝藏', desc='尼古拉斯凯奇'),
        Album(name='湄公河行动', desc='张涵予'),
        Album(name='西游记', desc='央视版'),
    )
```

如果在 `name` 和 `desc` 字段上搜索 `钢铁侠`，会返回哪些文档呢？

```python
    Album(name='钢铁战士', desc='蜘蛛侠'),
    Album(name='钢铁意志', desc='猪猪侠奇遇记'),
```

那么这 2 个文档如何排序呢？根据 solr 的评分规则，两个文档在 name 字段上的评分应该一样，而 desc 字段，由于 `蜘蛛侠` 分词后的词条数少于 `猪猪侠奇遇记`，那么其评分应该更高，这样综合下来，`钢铁战士` 的评分理应高于 `钢铁意志`

那么实际上是这样吗，看下标准查询的结果

```json
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"name,desc,score",
      "indent":"on",
      "q":"name:钢铁侠 desc:钢铁侠",
      "_":"1478137119591",
      "wt":"json"}},
  "response":{"numFound":2,"start":0,"maxScore":0.30110103,"docs":[
      {
        "name":"钢铁战士",
        "desc":"蜘蛛侠",
        "score":0.30110103},
      {
        "name":"钢铁意志",
        "desc":"猪猪侠奇遇记",
        "score":0.2843732}]
  }}
```

和预想的一样，注意 `q=name:钢铁侠 desc:钢铁侠`，在这个测试里没有创建 copy field

那么如果用 eDisMax 呢？再看下结果

```json
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"name,desc,score",
      "indent":"on",
      "q":"钢铁侠",
      "qf":"name desc",
      "_":"1478137119591",
      "wt":"json",
      "defType":"edismax"}},
  "response":{"numFound":2,"start":0,"maxScore":0.23656732,"docs":[
      {
        "name":"钢铁战士",
        "desc":"蜘蛛侠",
        "score":0.23656732},
      {
        "name":"钢铁意志",
        "desc":"猪猪侠奇遇记",
        "score":0.23656732}]
  }}
```

使用 eDisMax，注意 `qf=name desc`，2 个文档的评分竟然一样。。。

## tie

DisMax/eDisMax 的 tie 参数，用于指示在多个字段都匹配到搜索关键字时如何计算最终的得分，假设各个字段各自的评分是 $$s_1,s_2,\ldots,s_n$$，则其计算公式如下

$$
score = max(s_1, s_2, \ldots , s_n) + tie * (\sum(s_1, s_2, \ldots , s_n) - max(s_1, s_2, \ldots , s_n))
$$

`tie` 的取值范围是 `[0, 1]`，默认情况下，`tie = 0`，solr 建议 `tie=0.1`

## tie 的使用

知道了 DisMax/eDisMax 里是如何计算最终得分，那么就可以通过设置 tie 值来影响文档的得分

1. `tie=0` 评分最高的那个字段的得分就是文档的最终得分
2. `tie=1` 所有字段的得分相加就是文档的最终得分

还是上面的例子，来看一下 `tie=0.1` 的结果

```json
{
  "responseHeader":{
    "status":0,
    "QTime":2,
    "params":{
      "fl":"name,desc,score",
      "indent":"on",
      "tie":"0.1",
      "q":"name:钢铁侠 desc:钢铁侠",
      "_":"1478137119591",
      "wt":"json",
      "defType":"edismax"}},
  "response":{"numFound":2,"start":0,"maxScore":0.30110103,"docs":[
      {
        "name":"钢铁战士",
        "desc":"蜘蛛侠",
        "score":0.30110103},
      {
        "name":"钢铁意志",
        "desc":"猪猪侠奇遇记",
        "score":0.2843732}]
  }}
```

这样的结果才是符合我们预期的结果

> 注意

> 实际上这和标准查询解析器的评分是一致的。

## 结论

使用 DisMax/eDisMax 查询解析器时，如果在多个字段上搜索，应注意设置 `tie` 参数，solr 建议设置为 `tie=0.1`，否则查询结果排序可能不符合预期