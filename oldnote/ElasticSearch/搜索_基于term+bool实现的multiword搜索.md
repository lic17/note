---
title: 搜索_基于term+bool实现的multiword搜索
categories: elasticsearch   
toc: true  
tag: [elasticsearch]
---





1、普通match如何转换为term+should

```
{
    "match": { "title": "java elasticsearch"}
}
```

使用诸如上面的match query进行多值搜索的时候，es会在底层自动将这个match query转换为bool的语法
bool should，指定多个搜索词，同时使用term query

```
{
  "bool": {
    "should": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }}
    ]
  }
}
```

2、and match如何转换为term+must

```
{
    "match": {
        "title": {
            "query":    "java elasticsearch",
            "operator": "and"
        }
    }
}

{
  "bool": {
    "must": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }}
    ]
  }
}
```

3、minimum_should_match如何转换

```
{
    "match": {
        "title": {
            "query":                "java elasticsearch hadoop spark",
            "minimum_should_match": "75%"
        }
    }
}

{
  "bool": {
    "should": [
      { "term": { "title": "java" }},
      { "term": { "title": "elasticsearch"   }},
      { "term": { "title": "hadoop" }},
      { "term": { "title": "spark" }}
    ],
    "minimum_should_match": 3 
  }
}
```

上一讲，为啥要讲解两种实现multi-value搜索的方式呢？实际上，就是给这一讲进行铺垫的。match query --> bool + term。


搜索 term的同时，进行范围过滤

```
{
	"query":{
		"bool":{
			"must":[
				{"term":{"dstip":"223.252.199.69"}},	
				{"term":{"srcip":"172.16.115.35"}},
				{
					"range":{
						"starttime": {
							"gte":  1521136429
						}
					}
				},
				{
					"range":{
						"endtime": {
							"lt":   1521136439
						}
					}
				}
			]
		}
	}
}

```
