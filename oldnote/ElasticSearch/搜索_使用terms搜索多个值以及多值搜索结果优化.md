---
title: 搜索_使用terms搜索多个值以及多值搜索结果优化
categories: elasticsearch   
toc: true  
tag: [elasticsearch]
---





term: {"field": "value"}
terms: {"field": ["value1", "value2"]}

sql中的in

select * from tbl where col in ("value1", "value2")

1、为帖子数据增加tag字段

```
POST /forum/article/_bulk
{ "update": { "_id": "1"} }
{ "doc" : {"tag" : ["java", "hadoop"]} }
{ "update": { "_id": "2"} }
{ "doc" : {"tag" : ["java"]} }
{ "update": { "_id": "3"} }
{ "doc" : {"tag" : ["hadoop"]} }
{ "update": { "_id": "4"} }
{ "doc" : {"tag" : ["java", "elasticsearch"]} }
```

2、搜索articleID为KDKE-B-9947-#kL5或QQPX-R-3956-#aD8的帖子，搜索tag中包含java的帖子

```
GET /forum/article/_search 
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "articleID": [
            "KDKE-B-9947-#kL5",
            "QQPX-R-3956-#aD8"
          ]
        }
      }
    }
  }

GET /forum/article/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "terms" : { 
                    "tag" : ["java"]
                }
            }
        }
    }
}


  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "forum",
        "_type": "article",
        "_id": "2",
        "_score": 1,
        "_source": {
          "articleID": "KDKE-B-9947-#kL5",
          "userID": 1,
          "hidden": false,
          "postDate": "2017-01-02",
          "tag": [
            "java"
          ]
        }
      },
      {
        "_index": "forum",
        "_type": "article",
        "_id": "4",
        "_score": 1,
        "_source": {
          "articleID": "QQPX-R-3956-#aD8",
          "userID": 2,
          "hidden": true,
          "postDate": "2017-01-02",
          "tag": [
            "java",
            "elasticsearch"
          ]
        }
      },
      {
        "_index": "forum",
        "_type": "article",
        "_id": "1",
        "_score": 1,
        "_source": {
          "articleID": "XHDK-A-1293-#fJ3",
          "userID": 1,
          "hidden": false,
          "postDate": "2017-01-01",
          "tag": [
            "java",
            "hadoop"
          ]
        }
      }
    ]
  }
}


```


3、优化搜索结果，仅仅搜索tag只包含java的帖子

where tag in ('java') and tag_cnt=1

```

POST /forum/article/_bulk
{ "update": { "_id": "1"} }
{ "doc" : {"tag_cnt" : 2} }
{ "update": { "_id": "2"} }
{ "doc" : {"tag_cnt" : 1} }
{ "update": { "_id": "3"} }
{ "doc" : {"tag_cnt" : 1} }
{ "update": { "_id": "4"} }
{ "doc" : {"tag_cnt" : 2} }

GET /forum/article/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "term": {
                "tag_cnt": 1
              }
            },
            {
              "terms": {
                "tag": ["java"]
              }
            }
          ]
        }
      }
    }
  }
}

["java", "hadoop", "elasticsearch"]

```

4、学到的知识点梳理

（1）terms多值搜索
（2）优化terms多值搜索的结果
（3）相当于SQL中的in语句



