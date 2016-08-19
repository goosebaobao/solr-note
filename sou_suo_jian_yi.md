# 搜索建议

## 什么是搜索建议

如下图，在搜索框内输入`斗`，红框内展示的即为**搜索建议**

![搜索建议示意图](search-suggest.PNG)

所谓搜索建议，就是根据用户已输入的关键字，提供几个几个备选的关键字。Solr 自带建议组件，只需简单配置即可。

## 在 solr 里实现搜索建议

### 在 solrconfig.xml 添加建议组件

```xml
<searchComponent name="suggest" class="solr.SuggestComponent">
  <lst name="suggester">
  <str name="name">mySuggester</str>
  <str name="lookupImpl">FuzzyLookupFactory</str>
  <str name="dictionaryImpl">DocumentDictionaryFactory</str>
  <str name="field">name</str>
  <str name="weightField">heat</str>
  <str name="suggestAnalyzerFieldType">string</str>
  <str name="buildOnStartup">false</str>
  </lst>
</searchComponent>
```

参数说明

| 参数 | 说明 |
| -- | -- |
| searchComponent Name | 该建议组件的名称 |
| name | 建议器的名称，在 URL 参数和 SearchHandler 配置里引用该名称 |
| lookupImpl | 如何查找建议，solr 自带了几个可用的实现，默认为 JaspellLookupFactory |
| dictionaryImpl | 建议词典，solr 自带了几个词典的实现，默认的词典实现为 HighFrequencyDictionaryFactory |
| field | 索引里的一个字段，用来作为建议词条的基础 |
| weightField | 权重字段，用来对多个建议关键字进行排序 |
| buildOnStartup | true 表示在 Solr 启动时或 core 重载时创建词典。通常把这个设置设为 false，并手工创建建议，即在提交建议请求时使用 `suggest.build=true`|

### 在 solrconfig.xml 添加建议请求处理器

添加建议组件以后，必须在 `solrconfig.xml` 里添加请求处理器，示例如下

```xml
<requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy">
  <lst name="defaults">
    <str name="suggest">true</str>
    <str name="suggest.count">10</str>
  </lst>
  <arr name="components">
    <str>suggest</str>
  </arr>
</requestHandler>
```

参数说明

| 参数 | 说明 |
| -- | -- |
| suggest=true | 该参数必须为 true |
| suggest.dictionary | 在建议组件里配置的建议器的名称。该参数是必须滴，可以在请求处理器里设置，也可以在查询时通过参数传递 |
| suggest.q | 查找建议时用的查询 |
| suggest.count | 指定 Solr 返回的建议数量 |
| suggest.build | true 表示构建建议索引。仅在初次请求时有用 |
| components.name| 这里填入上一步骤定义的建议组件的名称 |

### 如何获取建议

```bash
http://localhost:8983/solr/tv/suggest?suggest=true&suggest.build=true \
&suggest.dictionary=mySuggester&wt=json&suggest.q=xiyou
```

## 实战

还是以之前建好的 tv 来实战

### solrconfig.xml 配置

在 solrconfig.xml 最后添加

```xml
<!-- suggestion -->
<searchComponent name="suggest" class="solr.SuggestComponent">
  <lst name="suggester">
    <!-- 建议器名为 tvSuggester -->
    <str name="name">tvSuggester</str>
    <!-- tvsuggest 字段用来提供建议 -->
    <str name="field">tvsuggest</str>
    <!-- heat 字段用来排序 -->
    <str name="weightField">heat</str>
    <str name="suggestAnalyzerFieldType">string</str>
    <str name="buildOnStartup">false</str>
  </lst>
</searchComponent>

<requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy">
  <lst name="defaults">
    <str name="suggest">true</str>
    <str name="suggest.count">10</str>
  </lst>
  <arr name="components">
    <str>suggest</str>
  </arr>
</requestHandler>
```

### 修改 shcma.xml

在建议组件的配置里，使用了 `tvsuggest` 字段，这个字段专用于提供搜索建议，需要在 `schema.xml` 里定义这个字段，如下

```xml
<fields>
		......

		<!-- 搜索建议，支持拼音和中文 -->
		<field name="tvsuggest" type="string" indexed="true" 
            multiValued="true" stored="false" />

</fields>

<copyField source="nameJianpin" dest="tvsuggest" />
<copyField source="nameQuanpin" dest="tvsuggest" />
<copyField source="name" dest="tvsuggest" />

......
```

新建一个 `tvsuggest` 字段，并将 `name`, `nameJianpin`, `nameQuanpin` 3 个字段复制到该字段

### 获取建议

使用 `西游` 作为关键字来获取建议

```bash
http://172.17.21.76:8983/solr/tv/suggest?suggest=true&suggest.build=true&\
suggest.dictionary=tvSuggester&wt=json&suggest.q=西游
```

返回结果

```json
{
	"responseHeader": {
		"status": 0,
		"QTime": 391
	},
	"command": "build",
	"suggest": {
		"tvSuggester": {
			"西游": {
				"numFound": 8,
				"suggestions": [{
					"term": "西游捕鱼(赢话费iPhone)",
					"weight": 1,
					"payload": ""
				},
				{
					"term": "西游漫记",
					"weight": 1,
					"payload": ""
				},
				{
					"term": "西游记1",
					"weight": 2,
					"payload": ""
				},
				{
					"term": "西游记2",
					"weight": 2,
					"payload": ""
				},
				{
					"term": "西游记3",
					"weight": 1,
					"payload": ""
				},
				{
					"term": "西游记之三件宝贝",
					"weight": 1,
					"payload": ""
				},
				{
					"term": "西游记之大闹天宫",
					"weight": 1,
					"payload": ""
				},
				{
					"term": "西游记（浙版）",
					"weight": 1,
					"payload": ""
				}]
			}
		}
	}
}
```

使用 `xiyou` 作为关键字来获取建议

```bash
http://172.17.21.76:8983/solr/tv/suggest?suggest=true&suggest.build=true&\
suggest.dictionary=tvSuggester&wt=json&suggest.q=xiyou
```

返回结果

```json
{
    "responseHeader": {
        "status": 0,
        "QTime": 306
    },
    "command": "build",
    "suggest": {
        "tvSuggester": {
            "xiyou": {
                "numFound": 9,
                "suggestions": [
                    {
                        "term": "Xiyoubuyu(yinghuafeiiPhone)",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyouji1",
                        "weight": 2,
                        "payload": ""
                    },
                    {
                        "term": "Xiyouji2",
                        "weight": 2,
                        "payload": ""
                    },
                    {
                        "term": "Xiyouji3",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyoujizheban",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyoujizhidanaotiangong",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyoujizhisanjianbaobei",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyoumanji",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xiyoutonghua",
                        "weight": 1,
                        "payload": ""
                    }
                ]
            }
        }
    }
}
```

这尼玛就有点尴尬了。。。。为神马返回的建议也是拼音了？

再用 `xy` 做关键字试下

```bash
http://172.17.21.76:8983/solr/tv/suggest?suggest=true&suggest.build=true&\
suggest.dictionary=tvSuggester&wt=json&suggest.q=xy
```

返回结果

```json
{
    "responseHeader": {
        "status": 0,
        "QTime": 307
    },
    "command": "build",
    "suggest": {
        "tvSuggester": {
            "xy": {
                "numFound": 10,
                "suggestions": [
                    {
                        "term": "Xy",
                        "weight": 2,
                        "payload": ""
                    },
                    {
                        "term": "Xy360sd",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xybmm",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xyby(yhfiPhone)",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xyctg",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xydh",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xydn",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xydstlagx",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xyeyhwq",
                        "weight": 1,
                        "payload": ""
                    },
                    {
                        "term": "Xyfsszwdqc",
                        "weight": 1,
                        "payload": ""
                    }
                ]
            }
        }
    }
}
```

无语了。。。

## 结论

一般搜索建议的需求应该是无论关键字是中文还是拼音，返回的搜索建议都是中文。但 solr 的建议组件很显然没有这么智能，在输入中文时获取中文建议无压力，但用拼音搜索时，想要返回中文的搜索建议还是有待深入研究滴
