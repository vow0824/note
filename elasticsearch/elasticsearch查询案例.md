###### elasticsearch查询案例

```json
PUT /cars

{
  "mappings" : {
      "transactions" : {
        "properties" : {
          "color" : {
            "type" : "keyword"
          },
          "make" : {
            "type" : "keyword"
          },
          "price" : {
            "type" : "long"
          },
          "sold" : {
            "type" : "date"
          }
        }
      }
    }
}

GET /cars/_mapping

POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }



GET /cars/_search
{
  "query": {
    "match_all": {}
  }
}

GET /cars/_search
{
  "query": {
    "match": {
      "color": "red"
    }
  }
}

GET /cars/_search
{
  "query": {
    "multi_match": {
      "query": "r",
      "fields": ["color","make"]
    }
  }
}

#复合查询
GET /cars/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "color": "red"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "price": "10000"
          }
        }
      ],
      "should": [
        {
          "match": {
            "make": "honda"
          }
        }
      ]
    }
  }
}


GET /cars/_search
{
  "query": {
    "bool": {
      "must": [
        {"range": {
          "price": {
            "gte": 20000,
            "lte": 80000
          }
        }}
      ]
    }
  },
  "size": 0
}

#区间查询
GET /cars/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "price": {
            "gte": 20000,
            "lte": 80000
          }
        }
      }
    }
  }
}

#聚合
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "colors": {
      "terms": {
        "field": "make.keyword"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

#范围聚合
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 2000,
        "min_doc_count": 1
      },
      "aggs": {
        "max_price": {
          "max": {
            "field": "price"
          }
        },
        "min_price": {
          "min": {
            "field": "price"
          }
        },
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}

#日期范围聚合
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "days": {
      "date_histogram": {
        "field": "sold",
        "interval": "90d",
        "format": "yyyy-MM-dd", 
        "min_doc_count": 1
      },
      "aggs": {
        "max_price": {
          "max": {
            "field": "price"
          }
        },
        "min_price": {
          "min": {
            "field": "price"
          }
        }
      }
    }
  }
}

#聚合中聚合
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "days": {
      "date_histogram": {
        "field": "sold",
        "interval": "1q",
        "format": "yyyy-MM-dd", 
        "min_doc_count": 1
      },
      "aggs": {
        "brands": {
          "terms": {
            "field": "make.keyword"
          },
          "aggs": {
            "sales": {
              "sum": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}


GET /cars/_search
{
  "size": 0,
  "query": {
    "term": {
      "make": {
        "value": "ford"
      }
    }
  }, 
  "aggs": {
    "ford_sales": {
      "sum": {
        "field": "price"
      }
    },
    "all": {
      "global": {}, 
      "aggs": {
        "all_sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

#global抛弃查询条件
GET /cars/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "term": {
          "make": "ford"
        }
      }
    }
  }, 
  "aggs": {
    "ford_sales": {
      "sum": {
        "field": "price"
      }
    },
    "all": {
      "global": {}, 
      "aggs": {
        "all_sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

#聚合中过滤
GET /cars/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": {
        "term": {
          "make": "ford"
        }
      }
    }
  }, 
  "aggs": {
    "sales": {
      "filter": {
        "term": {
          "color": "blue"
        }
      },
      "aggs": {
        "blue_sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

#聚合后排序
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "colors": {
      "terms": {
        "field": "color.keyword",
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}

#排序 
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 20000, 
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}


#按照metrics排序(metrics结果只有一个值)
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "sales_rank": {
      "terms": {
        "field": "make.keyword",
        "order": {
          "sales": "desc"
        }
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

#按照metrics排序(metrics结果有多个值)
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "sales_rank": {
      "terms": {
        "field": "make.keyword",
        "order": {
          "stat.avg": "desc"
        }
      },
      "aggs": {
        "stat": {
          "extended_stats": {
            "field": "price"
          }
        }
      }
    }
  }
}

```

