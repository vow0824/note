###### springboot整合elasticsearch

根据springboot版本或者es版本选择对应的jar包

![image-20230131143238180](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230131143238180.png)



引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

springboot简单配置

```yaml
spring:
  elasticsearch:
    rest:
      uris: 192.168.119.16:9200
```

在es中创建索引和数据（这里直接引用的官方测试数据）

```json
{
  "shakespeare" : {
    "mappings" : {
      "properties" : {
        "line_id" : {
          "type" : "long"
        },
        "line_number" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "play_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "speaker" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "speech_number" : {
          "type" : "long"
        },
        "text_entry" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

创建对应的实体类（必须要有Id属性）

```java
@Data
@Document(indexName = "shakespeare")
public class Shakespeare {

    @Id
    @Field(name = "line_id")
    private Long lineId;

    @Field(name = "line_number")
    private String lineNumber;

    @Field(name = "play_name")
    private String playName;

    @Field(name = "speaker")
    private String speaker;

    @Field(name = "speech_number")
    private Long speechNumber;

    @Field(name = "text_entry")
    private String textEntry;
}
```

创建查询的接口

```java
public interface ShakespeareDao extends ElasticsearchRepository<Shakespeare, Long> {
}
```

简单查询

```java
@Service
public class EsService {
    
    @Autowired
    private ShakespeareDao shakespeareDao;

    public List<Shakespeare> searchShakespeare() {
        Iterable<Shakespeare> all = shakespeareDao.findAll(PageRequest.of(1, 10));
        List<Shakespeare> list = new ArrayList<>();
        for (Shakespeare shakespeare : all) {
            list.add(shakespeare);
        }
        return list;
    }
}
```



