# 提高匹配度之加权查询

如果想要对查询结果按某个字段排序，可以用 `sort` 参数，但 `sort` 参数会忽视文档匹配度，这样可能会导致匹配度较低的文档位于搜索结果前列。

例如，有如下的文档

```python
    albums = (
        Album(name='钢铁侠', heat=30, type=1),
        Album(name='超人：钢铁之躯', heat=60, type=1),
        Album(name='超凡蜘蛛侠', heat=85, type=1),
        Album(name='神奇猪猪侠历险记', heat=99, type=2),
    )
```

按 `heat` 字段升序排列，会导致评分最高的文档排在最后，如下

```json
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "sort":"heat desc",
      "fl":"name,heat,score",
      "indent":"on",
      "q":"钢铁侠",
      "_":"1478050591433",
      "wt":"json"}},
  "response":{"numFound":4,"start":0,"maxScore":1.0189849,"docs":[
      {
        "heat":99,
        "name":"神奇猪猪侠历险记",
        "score":0.11500417},
      {
        "heat":85,
        "name":"超凡蜘蛛侠",
        "score":0.1533389},
      {
        "heat":60,
        "name":"超人：钢铁之躯",
        "score":0.25425506},
      {
        "heat":30,
        "name":"钢铁侠",
        "score":1.0189849}]
  }}
```

可见，简单的按某个字段升序或降序排列并不足以提高匹配度。

考虑这样一个需求：搜索某个新闻事件(例如美国大选邮件门)，其排序应该是综合考虑关键字匹配度和新闻报道的时间，匹配度 100% 但是时间过去很久，也比不上最近发生但匹配度稍差的新闻。

DixMax/eDisMax 提供了 `bq` 参数，也许可以满足这样的需求

## bq

`bq` 参数提供了一个子查询，满足该子查询的文档其匹配度会提升。如果有多个条件都要加权，可以传递多个 `bq` 参数，其格式如下

```
bq=查询1&b1=查询2&...&bq=查询n
```

先看下不支持 `bq` 参数的标准查询解析器的结果

```json


{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"name,heat,score",
      "indent":"on",
      "q":"钢铁侠",
      "_":"1478050591433",
      "wt":"json"}},
  "response":{"numFound":4,"start":0,"maxScore":1.0189849,"docs":[
      {
        "heat":30,
        "name":"钢铁侠",
        "score":1.0189849},
      {
        "heat":60,
        "name":"超人：钢铁之躯",
        "score":0.25425506},
      {
        "heat":85,
        "name":"超凡蜘蛛侠",
        "score":0.1533389},
      {
        "heat":99,
        "name":"神奇猪猪侠历险记",
        "score":0.11500417}]
  }}
```

对热度较高的文档加权，即 `heat>=80`

```json
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"name,heat,score",
      "bq":"heat:[80 TO *]",
      "indent":"on",
      "q":"钢铁侠",
      "_":"1478050591433",
      "wt":"json",
      "defType":"edismax"}},
  "response":{"numFound":4,"start":0,"maxScore":0.86861277,"docs":[
      {
        "heat":30,
        "name":"钢铁侠",
        "score":0.86861277},
      {
        "heat":85,
        "name":"超凡蜘蛛侠",
        "score":0.65355295},
      {
        "heat":99,
        "name":"神奇猪猪侠历险记",
        "score":0.6208753},
      {
        "heat":60,
        "name":"超人：钢铁之躯",
        "score":0.21673451}]
  }}
```

可以看到

1. `钢铁侠` 因为热度较低未得到加权，但由于匹配度高，结果依然排第一
2. `超人：钢铁之躯` 由于匹配度并无明显优势，又不是高热度未获得加权，从第二名跌到榜尾

再换个条件试一下，这次试试能否将原本排在榜尾的 `神奇猪猪侠历险记` 往前排，加权条件为 `heat=99`

```
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"name,heat,score",
      "bq":"heat:99",
      "indent":"on",
      "q":"钢铁侠",
      "_":"1478050591433",
      "wt":"json",
      "defType":"edismax"}},
  "response":{"numFound":4,"start":0,"maxScore":0.86861277,"docs":[
      {
        "heat":30,
        "name":"钢铁侠",
        "score":0.86861277},
      {
        "heat":99,
        "name":"神奇猪猪侠历险记",
        "score":0.6208753},
      {
        "heat":60,
        "name":"超人：钢铁之躯",
        "score":0.21673451},
      {
        "heat":85,
        "name":"超凡蜘蛛侠",
        "score":0.13071059}]
  }}
```

嗯，这个结果还不错的，`神奇猪猪侠历险记` 虽然提前了，但依然没有影响 `钢铁侠` 的榜首地位。

## 多重 bq

这一次试下多重 `bq`，即 `bq=heat:99&bq=type:2`，只有 `神奇猪猪侠历险记` 同时满足这 2 个条件，能否则威胁 `钢铁侠` 的榜首地位呢

查询 url

```
view-source:http://dev:8983/solr/tv/select?bq=heat:99&defType=edismax&fl=name,heat,score&indent=on&q=%E9%92%A2%E9%93%81%E4%BE%A0&wt=json&bq=type:2
```

查询结果

```json
{
  "responseHeader":{
    "status":0,
    "QTime":2,
    "params":{
      "fl":"name,heat,score",
      "bq":["heat:99",
        "type:2"],
      "indent":"on",
      "q":"钢铁侠",
      "wt":"json",
      "defType":"edismax"}},
  "response":{"numFound":4,"start":0,"maxScore":1.0135437,"docs":[
      {
        "heat":99,
        "name":"神奇猪猪侠历险记",
        "score":1.0135437},
      {
        "heat":30,
        "name":"钢铁侠",
        "score":0.7697503},
      {
        "heat":60,
        "name":"超人：钢铁之躯",
        "score":0.19206655},
      {
        "heat":85,
        "name":"超凡蜘蛛侠",
        "score":0.11583357}]
  }}
```

啊哈，`神奇猪猪侠历险记` 终于排第一啦

## 结论

1. `bq` 参数并不影响查询的结果，影响的是结果的排序
2. `bq` 的加权是有限的，并不足以影响匹配度相差较大的文档之间的相对排序
3. 越多的 `bq` 加权，匹配度提升就越明显