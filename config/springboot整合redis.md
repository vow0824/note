###### springboot整合redis

简单整合redis

引入redis-starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

简单配置

```yaml
spring:
  redis:
    host: 192.168.119.16
    port: 6379
```

测试代码

```java
@Service
public class RedisService {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    public void set() {
        stringRedisTemplate.opsForValue().set("a", "b");
    }

    public String get() {
        return stringRedisTemplate.opsForValue().get("a");
    }
}
```





简单整合redisson

引入依赖，排除默认的lettuce客户端，使用jedis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.19.2</version>
</dependency>
```

测试代码

```java
@Service
@Slf4j
public class RedisService {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private RedissonClient redisson;

    private static final String HELLO_WORLD_LOCK = "hello_world";

    public void set() {
        stringRedisTemplate.opsForValue().set("a", "b");
    }

    public String get() {
        return stringRedisTemplate.opsForValue().get("a");
    }

    public void printHelloWorld() {
        RLock lock = redisson.getLock(HELLO_WORLD_LOCK);
        lock.lock(2, TimeUnit.SECONDS);
        log.info("hello world!");
        lock.unlock();
    }
}
```

