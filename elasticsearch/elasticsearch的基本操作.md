###### Elasticsearch 的基本操作

1、_cat

GET/_cat/nodes：查看所有节点

GET/_cat/health：查看es健康状况

GET/_cat/master：查看主节点

GET/_cat/indices：查看所有索引

2、索引一个文档（保存）

保存一个数据，保存在哪个索引的哪个类型下，指定用哪个唯一标识

PUT costomer/external/1，在customer索引下的external类型下保存1号数据为

```json
PUT customer/external/1
{
  "name":"John Doe"
}
```



PUT和POST都可以

POST新增，如果不指定id，会自动生成id，指定id就会修改这个数据，并新增版本号

PUT可以新增可以修改。PUT必须指定id，由于PUT需要只从id，我们一般都用来修改操作，不指定id会报错。

3、查询文档

GET customer/external/1

结果：

{

  "_index": "customer",   //在哪个索引

  "_type": "external",   在哪个类型

  "_id": "1",       //记录id

  "_version": 1,     //版本号

  "_seq_no": 0,     //并发控制字段，每次更新就会+1，用来做乐观锁

  "_primary_term": 1,   //，同上，主分片重新分配，如重启，就会变化

  "found": **true**,

  "_source": {       //真正的内容

​    "name": "Jhon Doe"

  }

}

更新携带   ?if_seq_no=0&if_primary_term=1

4、更新文档

```json
POST customer/external/1/_update

{
  "doc":{
     "name":"John Doe1"
   }
}
```



或者

```json
POST customer/external/1
{
  "name":"John Doe2"
}
```



或者

```json
PUT customer/external/1
{
  "name":"John Doe3"
}
```



不同：POST操作会对比源文档数据，如果相同不会有什么操作，文档version不增加PUT操作总会将数据重新保存并增加version版本：

  带_update对比元数据如果一样就不进行任何操作。

  看场景：

​     对于大并发更新，不带_update;

​     对于大并发查询偶尔更新，带_update；对比更新，重新计算分配规则

更新同时增加属性

```json
POST customer/external/1/_update
{
  "doc":{
     "name":"John",
     "age":20
   }
}
```



PUT和POST不带_update也可以

5、删除文档&索引

DELETE customer/external/1

DELETE customer

6、bulk批量API

```json
POST customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"Jane Doe"}
```



语法格式

{action: {metadata}}\n

{request body   }\n

{action:{metadata }}\n

{request body   }\n

复杂实例：

```json
POST/_bulk
{"delete":{"_index":"website","_type":"blog","_id":"123"}}
{"create":{"_index":"website","_type":"blog","_id":"123"}}
{"title":"My first blog post"}
{"index":{"_index":"website","_type":"blog"}}
{"title":"My second blog post"}
{"update":{"_index":"website","_type":"blog","_id":""123"}}
{"doc":{"title":"My upload blog post"}}
```



 

bulk API以此按顺序执行所有的action（动作）。如果一个单个的动作因任何原因而失败，它将继续处理它后面剩余的动作。当bulk API返回时，它将提供每个动作的状态（与发送的顺序相同），所以我们可以检查是否一个指定的动作是不是失败了。