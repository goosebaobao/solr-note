# 搜索建议

## 什么是搜索建议

如下图，在搜索框内输入`斗`，红框内展示的即为**搜索建议**

![搜索建议示意图](search-suggest.PNG)

所谓搜索建议，就是根据用户已输入的关键字，提供几个几个备选的关键字。Solr 自带建议组件，只需简单配置即可。

## 如何配置

### 配置 solrconfig.xml

在 `solrconfig.xml`里添加建议组件，如下

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