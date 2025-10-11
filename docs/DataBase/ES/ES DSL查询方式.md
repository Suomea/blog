查询所有：查询出所有数据，一般测试使用，match_all，不会返回全部数据，有限制条数啦。
全文检索：利用分词器对查询文本进行分词，然后匹配倒排索引库，然后拿到文档的 ID，计算得分，根据 ID 查询文档。
- match_query
- match
- multi_match_query
精确查询：根据精确的词条查找数据，一般查询 keyword、数值、日期、boolean 等类型字段。
- ids
- range
- term
地理查询：根据经纬度查询。
- geo_distance
- geo_bounding_box
复合查询：将上述的查询条件组合起来，合并查询条件。
- bool
- function_score

基本语法
```
GET /indexName/_search
{
	"query": {
		"查询类型": {
			"查询条件": "条件值"
		}
	}
}
```

查询示例
```
# 查询全部
GET /book/_search
{
  "query": {
    "match_all": {}
  }
}

# 全文检索，单字段
GET /book/_search
{
  "query": {
    "match": {
      "all": "Java"
    }
  }
}

# 全文检索，多字段，可以使用 * 表示全部字段
GET /book/_search
{
  "query": {
    "multi_match": {
      "query": "Java",
      "fields": ["name", "author"]
    }
  }
}

# 精确查询词条查询
GET /book/_search
{
  "query": {
    "term": {
      "name": {
        "value": "Java 揭秘"
      }
    }
  }
}

# 精确查询范围查询
GET /boot/_search
{
  "query": {
    "range": {
      "public_year": {
        "gte": 2013,
        "lte": 2034
      }
    }
  }
}




```