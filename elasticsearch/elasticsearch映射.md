###### elasticsearch映射

1、创建映射

```json
PUT   http://192.168.43.219:9200/my_index
{
  "mappings":{
    "properties":{
      "age":{
        "type":"integer"
      },
      "email":{"type":"keyword"},
      "name":{"type":"text"}
    }
  }
}
```



2、添加新的字段映射

```json
PUT   http://192.168.43.219:9200/my_index/_mapping
{
  "properties":{
    "employee-id":{
      "type":"keyword",
      "indwx":**false**
    }
  }
}
```



3、更新映射

对于已经存在的映射字段，我们不能更新，更新必须创建新的索引进行数据迁移

4、数据迁移

先创建出正确的映射

```json
PUT   http://192.168.43.219:9200/newbank/_mapping

{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "text"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "state": {
        "type": "keyword"
      }
    }
  }
}

POST  _reindex（固定写法）
{
  "source":{
     "index":"twitter"
   },
  "dest":{
     "index":"new_twitter"
   }
}
```



将就索引的type下的数据进行迁移

```json
POS  _reindex
{
  "source":{
     "index":"twitter",
     "type":"tweet"
   },
  "dest":{
     "index":"new_twitter"
   }
}
{
  "source":{
    "index":"bank",
    "type":"account"
  },
  "dest":{
    "index":"newbank"
  }
}
```

