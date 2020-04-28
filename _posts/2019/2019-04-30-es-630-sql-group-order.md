---
title: elasticsearch v6.3.0 sql group-order
tags: es
key: 29
modify_date: 2019-04-30 18:00:00 +08:00
---

----
# Overview
es v6.3.0之后，推出了es-SQL的支持。今天来试试这个功能。

----
# 测试数据集
![geonames](https://upload-images.jianshu.io/upload_images/2189341-2115a1bf31d44a52.png)

----
# 简单语句
在简单语句的情况下，这个功能ok，具体表现如下，

![simple es-sql](https://upload-images.jianshu.io/upload_images/2189341-ee95232341e5f037.png)

```
# execute
curl -X POST "$HOST/_xpack/sql?format=txt" -H 'Content-Type: application/json' -d'
{
    "query": "SELECT * FROM bm ORDER BY longitude DESC limit 3",
    "fetch_size": 3
}'

# translate to es DSL
curl -X POST "$HOST/_xpack/sql/translate?pretty" -H 'Content-Type: application/json' -d'
{
    "query": "SELECT * FROM bm ORDER BY longitude DESC limit 3",
    "fetch_size": 3
}'

# execute2（双引号里面的字符串）
curl -X POST "$HOST/_xpack/sql?format=txt" -H 'Content-Type: application/json' -d"
{
    \"query\": \"SELECT country_code, population AS sum_pop FROM bm WHERE population > 1 AND country_code = 'CN' ORDER BY population DESC\",
    \"fetch_size\": 11
}"

```
![translate from es DSL](https://upload-images.jianshu.io/upload_images/2189341-ad4f8d9bc699fda7.png)


# 稍复杂语句
## mysql
我们先看在mysql数据库下面，这些复杂语句的**语法准确性**。
![mysql-process](https://upload-images.jianshu.io/upload_images/2189341-702b846b2af16a22.png)

## es-sql
### only group by
![only group by](https://upload-images.jianshu.io/upload_images/2189341-dd67963c24243b30.png)

![translate from es DSL与execute的返回结果一致](https://upload-images.jianshu.io/upload_images/2189341-aace9f1ea568ca44.png)

### group by with order by
当在`group by`之后添加`order by`，es-sql就不能正常解析了。而在es-DSL里面是可以实现这个agg-sort[功能](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/search-aggregations-pipeline-bucket-sort-aggregation.html)的。

![es-sql fail with group-order](https://upload-images.jianshu.io/upload_images/2189341-5b418ca1b3090177.png)

根据上一节的without order by解析出来的DSL，再配合agg-sort这个功能，来实现group-order。
![without order](https://upload-images.jianshu.io/upload_images/2189341-ba486f3c0819baa6.png)

![with order](https://upload-images.jianshu.io/upload_images/2189341-beeb5ebcb2b01d5e.png)


```
# without order
curl -X POST "$HOST/bm/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size" : 0,
  "query" : {
    "range" : {
      "population" : {
        "from" : 0,
        "to" : null,
        "include_lower" : true,
        "include_upper" : false,
        "boost" : 1.0
      }
    }
  },
  "_source" : false,
  "stored_fields" : "_none_",
  "aggregations" : {
    "groupby" : {
      "composite" : {
        "size" : 11,
        "sources" : [
          {
            "1674" : {
              "terms" : {
                "field" : "country_code",
                "order" : "asc"
              }
            }
          }
        ]
      },
      "aggregations" : {
        "1683" : {
          "sum" : {
            "field" : "population"
          }
        }
      }
    }
  }
}
'

# with order
curl -X POST "$HOST/bm/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size" : 0,
  "query" : {
    "range" : {
      "population" : {
        "from" : 0,
        "to" : null,
        "include_lower" : true,
        "include_upper" : false,
        "boost" : 1.0
      }
    }
  },
  "_source" : false,
  "stored_fields" : "_none_",
  "aggregations" : {
    "groupby" : {
      "composite" : {
        "size" : 11,
        "sources" : [
          {
            "1674" : {
              "terms" : {
                "field" : "country_code",
                "order" : "asc"
              }
            }
          }
        ]
      },
      "aggregations" : {
        "1683" : {
          "sum" : {
            "field" : "population"
          }
        }
        ,"population_bucket_sort": {
            "bucket_sort": {
                "sort": [
                  {"1683": {"order": "desc"}}
                ]
            }
        }
      }
    }
  }
}
'
```

----
# Others
![es-sql source code](https://upload-images.jianshu.io/upload_images/2189341-588c27b87baa8490.png)

不知道这个fix/enhancement是否可以在es-string通过antlr义成AST的es-DSL。有时间再回头看这个[issue](https://github.com/elastic/elasticsearch/issues/29965)。

costin回复说[Bucket Sort Aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/search-aggregations-pipeline-bucket-sort-aggregation.html)只是局部排序，非全局排序。但是至于如何实现全局排序，我仍然没有弄明白。

![costin reply](https://upload-images.jianshu.io/upload_images/2189341-e04dec1a83fc2359.png)
