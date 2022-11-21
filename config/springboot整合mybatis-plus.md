###### springboot整合mybatis-plus

mybatis-plus 3.1版本以上



导入依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.2</version>
</dependency>
<!-- 需要引入模板引擎 -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.30</version>
</dependency>
```



快速生成基本的mapper，entity

```java
public static void generate() {
        FastAutoGenerator.create("jdbc:mysql://192.168.136.16:3306/test", "root", "root")
                .globalConfig(builder -> {
                    builder.author("vow") // 设置作者
                            //.enableSwagger() // 开启 swagger 模式
                            .fileOverride() // 覆盖已生成文件
                            .outputDir("D:\\IdeaProjects\\demo\\src\\main\\java"); // 指定输出目录
                })
                .packageConfig(builder -> {
                    builder.parent("com.vow.demo.entity") // 设置父包名
                            .moduleName("user") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.xml, "D:\\IdeaProjects\\demo\\src\\main\\resources\\mapper")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("user");
                })
                .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
                .execute();
    }
```

