###### 简单的检索

Elasticsearch 检索及聚合

ES支持两种基方式检索

1.一个时通过使用REST request URI 发送检索参数（uri + 检索参数）

2.另一个时通过使用REST request body 来发送给他们（uti + 请求体）

1.检索信息

GET bank/_search  检索bank下所有信息，包括type和docs

GET bank/_search?q=*&sort=account_number:asc  请求参数方式检索

响应结果解释

took：Elasticsearch执行搜索的时间（毫秒）

time_out：告诉我们是否超时

_shares：告诉我们多少个分片被搜索了，以及统计了成功/失败的搜索分片

hits：搜索结果

hits.total：搜索结果

hits.hits：实际的搜索结果数组（默认为前10的文档）

sort：结果的排序key（键）（没有则按score排序）

score和max_score：相关性得分和最高得分（全文检索用）

uri + 请求体进行检索

```json
GET bank/_search

{
  "query":{
     "match_all":{}
   },
  "sort":[
     {
       "accornt_number":{
         "order":"desc"
       }
     }
   ]
}
```



HTTP客户端工具（POSTMAN），get请求不能携带请求体，我们变为post也是一样的

我们post一个json风格的查询请求体到 _search API

需要了解，一旦搜索的结果被返回，Elasticsearch就完成了这次请求，并且不会维护任何服务端的资源或者结果的cursor（游标）

2.Query DSI

基本语法格式

Elasticsearch提供了一个可以执行查询的Json风格的DSL（domain-specific language 领域特定语言）。这个被称为Query DSL.该查询语言非常全面，并且刚开始的时候感觉有点复杂。

一个查询语句的典型结构

{

  QUERY_NAME:{

​     ARGUEMENT:VALUE,

​     ARGUEMENT:VALUE...

   }

}

如果是针对某个字段，那么它的结构如下

```json
{
  QUERY_NAME:{
     FIELD_NAME:{
       ARGUEMENT:VALUE,
       ARGUEMENT:VALUE,
       ......
     }
   }
}

http://192.168.43.219:9200/bank/account/_search
{
  "query":{
    "match_all":{}
  },
  "sort":[
    {
      "balance":{
        "order":"desc"
      }
    }
  ],
  "from":5,
  "size":5,
  "_source":["balance", "firstname"]
}
```



match（匹配查询）

基本类型（非字符串），精确匹配

```json
GET bank/_search

{
  "query":{
     "match":{
       "account_number":"20"
     }
   }
}

http://192.168.43.219:9200/bank/account/_search
{
  "query":{
    "match":{
      "address":"Kings"
    }
  }
}

{
  "query":{
    "match":{
      "address":"mill road"
    }
  }
}
```



最终查询出address中包含mill或者road的所有记录，并给出相关性得分

短语匹配

```jso
{
  "query":{
    "match_phrase":{
      "address":"mill road"
    }
  }
}
```



查出address中包含mill road的所有记录，并给出相关性得分

multi_match（多字段匹配）

```json
{
  "query":{
    "multi_match":{
      "query":"mill",
      "fields":["state","address"]
    }
  }
}
```



state和address包含的都会查出来

bool（复合查询）

```json
{
  "query":{
    "bool":{
      "must":[
        {
          "match":{
            "gender":"F"
          }
        },
        {"match":{
          "address":"mill"
        }}
      ],
      "must_not":[
        {
          "match":{
            "age":"38"
          }
        }
      ],
      "should":[
        {
          "match":{
            "lastname":"Wallace"
          }
        }
      ]
    }
  }
}

filter（结果过滤）
{
  "query":{
    "bool":{
      "must":[
        {
          "range":{
            "age":{
              "gte":18,
              "lte":28
            }
          }
        }
      ]
    }
  }
}

{
  "query":{
    "bool":{
      "filter":[
        {
          "range":{
            "age":{
              "gte":18,
              "lte":28
            }
          }
        }
      ]
    }
  }
}

{
  "query":{
    "bool":{
      "must":[
        {
          "match":{
            "gender":"F"
          }
        },
        {"match":{
          "address":"mill"
        }}
      ],
      "must_not":[
        {
          "match":{
            "age":"38"
          }
        }
      ],
      "should":[
        {
          "match":{
            "lastname":"Wallace"
          }
        }
      ],
      "filter":{
          "range":{
            "age":{
              "gte":18,
              "lte":28
            }
          }
      }
    }
  }
}
```



term

和match一样，匹配某个属性的值，全文检索字段用match，非其他text字段匹配用term。

```json
{
  "query":{
    "term":{
      "age":"18"
    }      
  }
}
{
  "query":{
    "match_phrase":{
      "address":"mill road"
    }      
  }
}
{
  "query":{
    "match":{
      "address.keyword":"mill road"
    }      
  }
}
```



.keyword精确匹配

aggregation（执行聚合）

  聚合提供了从数据中分组和提取数据的能力。最简单的聚合方法大致等于SQL GROUP BY和SQL聚合函数。在Elasticsearch中您有执行搜索返回hits（命中结果），并且同时返回聚合结果，把一个响应中的所有hits（命中结果）分隔开的能力。这是非常强大且有效的，您可以指定查询和多个聚合，并且在依次使用中得到各自的（任何一个的）返回结果，使用一次简洁和简化的API来避免网络往返。

搜索address中包含mill的所有人的年龄分布以及平均年龄，但不显示这些人的详情

```json
{
  "query":{
    "match":{
      "address":"mill"
    }      
  },
  "aggs":{
    "ageAgg":{
      "terms":{
        "field":"age",
        "size":10
      }
    },
    "ageAvg":{
      "avg":{
        "field":"age"
      }
    },
    "balanceAvg":{
      "avg":{
        "field":"balance"
      }
    }
 }
}
```



size:0  不显示搜索数据

```json
{
  "query":{
    "match":{
      "address":"mill"
    }      
  },
  "aggs":{
    "ageAgg":{
      "terms":{
        "field":"age",
        "size":10
      }
    },
    "ageAvg":{
      "avg":{
        "field":"age"
      }
    },
    "balanceAvg":{
      "avg":{
        "field":"balance"
      }
    }
  },
  "size":0
}
```

按照年龄聚合，并且请求这些年龄的这些人的平均薪资

```json
{
 "query":{
    "match_all":{}      
  },
  "aggs":{
    "ageAgg":{
      "terms":{
        "field":"age",
        "size":100
      },
      "aggs":{
        "ageAvg":{
          "avg":{
            "field":"balance"
          }
        }
      }
    }
  }
}
```



查出所有年龄分布，并且这些年龄中M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资

```json
{
  "query":{
    "match_all":{}      
  },
  "aggs":{
    "ageAgg":{
      "terms":{
        "field":"age",
        "size":100
      },
      "aggs":{
        "genderAgg":{
          "terms":{
            "field":"gender.keyword",
            "size":10
          },
          "aggs":{
            "balanceAvg":{
              "avg":{
                "field":"balance"
              }
            }
          }
        },
        "aggBalanceAvg":{
          "avg":{
            "field":"balance"
          }
        }
     }
    }
  }
}
```

