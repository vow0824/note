###### springboot简单整合shardingsphere实现分库分表

数据库目录结构

![image-20230201111459971](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230201111459971.png)

代码目录结构

![image-20230201111635154](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230201111635154.png)

引入依赖，版本不一样，配置可能存在略微差别

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-core</artifactId>
    <version>4.1.1</version>
</dependency>
```

修改配置文件，此前设置的druid数据源配置可能需要修改，根据自己的配置来

```yaml
spring:
  datasource:
    main:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://192.168.119.16:3306/test
      username: root
      password: root
      #type: com.alibaba.druid.pool.DruidDataSource
    shard:
      url-0: jdbc:mysql://192.168.119.16:3306/test_0
      url-1: jdbc:mysql://192.168.119.16:3306/test_1
      username: root
      password: root
      initial-size: 3
      min-idle: 3
      max-active: 100

# 这里连接池的配置放在的代码配置中，让yaml配置文件看起来简单整洁一点
      #druid
#      initialSize: 5
#      minIdle: 5
#      maxActive: 20
#      maxWait: 60000
#      timeBetweenEvictionRunsMillis: 60000
#      minEvictableIdleTimeMillis: 300000
#      validationQuery: SELECT 1 FROM DUAL
#      testWhileIdle: true
#      testOnBorrow: false
#      testOnReturn: false
#      poolPreparedStatements: true
#
#      filters: stat,wall,log4j
#      maxPoolPreparedStatementPerConnectionSize: 20
#      useGlobalDataSourceStat: true
```

主数据源配置类

```java
@Data
@Configuration
public class DruidDataSourceConfig {

    private String url;
    private String username;
    private String password;
    private String driverClassName;
    private Integer initialSize;
    private Integer minIdle;
    private Integer maxActive;
    private Integer maxWait;
    private Integer timeBetweenEvictionRunsMillis;
    private Integer minEvictableIdleTimeMillis;
    private String validationQuery;
    private Boolean testWhileIdle;
    private Boolean testOnBorrow;
    private Boolean testOnReturn;
    private Boolean poolPreparedStatements;
    private Integer maxPoolPreparedStatementPerConnectionSize;
    private String filters;
    private String connectionProperties;
    private String xy;
    private String driver;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.main")
    public DruidDataSourceConfig mainDruidDataSourceConfig() {
        return new DruidDataSourceConfig();
    }

    @Bean
    @Primary
    public DataSource mainDataSource(@Qualifier("mainDruidDataSourceConfig") DruidDataSourceConfig config) {
        return dataSourceFactory(config);
    }

    public static DataSource dataSourceFactory(DruidDataSourceConfig config) {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(config.url);
        datasource.setUsername(config.username);
        datasource.setPassword(config.password);
        if (StringUtils.isEmpty(config.driver)) {
            datasource.setDriverClassName("com.mysql.jdbc.Driver");
        } else {
            datasource.setDriverClassName(config.driver);
        }
        datasource.setInitialSize(3);
        datasource.setMinIdle(3);
        datasource.setMaxActive(100);
        datasource.setMaxWait(300);
        datasource.setTimeBetweenEvictionRunsMillis(60000);
        datasource.setMinEvictableIdleTimeMillis(300000);
        datasource.setValidationQuery("SELECT 'x'");
        datasource.setTestWhileIdle(true);
        datasource.setTestOnBorrow(false);
        datasource.setTestOnReturn(false);
        datasource.setPoolPreparedStatements(false);
        datasource.setMaxPoolPreparedStatementPerConnectionSize(20);
        return datasource;
    }
}
```

主数据源mybatis配置

```java
@Configuration
public class MyBatisConfig {

    @Bean(name = "mainSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("mainDataSource") DataSource dataSource, MybatisPlusInterceptor interceptor) throws Exception {
        MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        sessionFactory.setMapperLocations(resolver.getResources("classpath*:mybatis/main/*Mapper.xml"));
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setPlugins(interceptor);
        return sessionFactory.getObject();
    }

    @Bean(name = "mainSqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("mainSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Bean(name = "mainMapperScannerConfigurer")
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer configurer = new MapperScannerConfigurer();
        configurer.setBasePackage("com.vow.demo.domain.main.*.mapper");
        configurer.setSqlSessionTemplateBeanName("mainSqlSessionTemplate");
        return configurer;
    }

    @Bean(name = "mainDataSourceTransactionManager")
    public DataSourceTransactionManager dataSourceTransactionManager(@Qualifier("mainDataSource") DataSource dataSource) {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }

    /**
     * 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //向Mybatis过滤器链中添加分页拦截器
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        //还可以添加i他的拦截器
        return interceptor;
    }

    /*@Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setUseDeprecatedExecutor(false);
    }*/
}
```

分库分表配置

```java
@Configuration
public class ShardingDataSourceConfig {

    @Bean(name = "shardDataSource")
    public DataSource getShardingDataSource(Environment env) throws SQLException {
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();

        shardingRuleConfig.getBindingTableGroups().add("app_order");
        shardingRuleConfig.getTableRuleConfigs().add(getTableRuleConfiguration("app_order", "order_id", "test_${0..1}.app_order_${0..1}"));

        shardingRuleConfig.setDefaultDatabaseShardingStrategyConfig(new StandardShardingStrategyConfiguration("uid", (PreciseShardingAlgorithm<Long>) (availableTargetNames, shardingValue) -> {
            String shard = "_" + (shardingValue.getValue() % 50 / 10 + 1);
            return availableTargetNames.stream().filter(name -> name.endsWith(shard)).findFirst().orElseThrow(IllegalArgumentException::new);
        }));

        shardingRuleConfig.setDefaultTableShardingStrategyConfig(new StandardShardingStrategyConfiguration("uid", (PreciseShardingAlgorithm<Long>) (availableTargetNames, shardingValue) -> {
            String shard = "_" + (shardingValue.getValue() % 10 + 1);
            return availableTargetNames.stream().filter(name -> name.endsWith(shard)).findFirst().orElseThrow(IllegalArgumentException::new);
        }));
        shardingRuleConfig.setDefaultDataSourceName("test_0");
        Properties properties = new Properties();
        properties.setProperty(ConfigurationPropertyKey.CHECK_TABLE_METADATA_ENABLED.getKey(), "false");
        properties.setProperty(ConfigurationPropertyKey.MAX_CONNECTIONS_SIZE_PER_QUERY.getKey(), "20");
        return ShardingDataSourceFactory.createDataSource(createDataSourceMap(env), shardingRuleConfig, properties);
    }

    private Map<String, DataSource> createDataSourceMap(Environment env) {
        Map<String, DataSource> result = new HashMap<>(5);
        for (int shard = 0; shard <= 1; shard++) {
            DataSource dataSource = createDataSource(env, shard);
            result.put("test_" + shard, dataSource);
        }
        return result;
    }

    private DataSource createDataSource(Environment env, int shard) {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(env.getRequiredProperty("spring.datasource.shard.url-" + shard));
        druidDataSource.setUsername(env.getRequiredProperty("spring.datasource.shard.username"));
        druidDataSource.setPassword(env.getRequiredProperty("spring.datasource.shard.password"));
        druidDataSource.setInitialSize(env.getRequiredProperty("spring.datasource.shard.initial-size", Integer.class));
        druidDataSource.setMinIdle(env.getRequiredProperty("spring.datasource.shard.min-idle", Integer.class));
        druidDataSource.setMaxActive(env.getRequiredProperty("spring.datasource.shard.max-active", Integer.class));
        druidDataSource.setMaxWait(60000);
        druidDataSource.setTimeBetweenEvictionRunsMillis(60000);
        druidDataSource.setMinEvictableIdleTimeMillis(300000);
        druidDataSource.setValidationQuery("SELECT 'x'");
        druidDataSource.setTestWhileIdle(true);
        druidDataSource.setTestOnBorrow(false);
        druidDataSource.setTestOnReturn(false);
        druidDataSource.setPoolPreparedStatements(false);
        druidDataSource.setMaxPoolPreparedStatementPerConnectionSize(20);
        return druidDataSource;
    }

    @Bean(name = "shardSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("shardDataSource") DataSource dataSource, MybatisPlusInterceptor interceptor) throws Exception {
        MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        sessionFactory.setMapperLocations(resolver.getResources("classpath*:mybatis/shard/*Mapper.xml"));
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setPlugins(interceptor);
        return sessionFactory.getObject();
    }

    @Bean(name = "shardSqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("shardSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Bean(name = "shardMapperScannerConfigurer")
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer configurer = new MapperScannerConfigurer();
        configurer.setBasePackage("com.vow.demo.domain.shard.*.mapper");
        configurer.setSqlSessionTemplateBeanName("shardSqlSessionTemplate");
        return configurer;
    }

    @Bean(name = "shardDataSourceTransactionManager")
    public DataSourceTransactionManager dataSourceTransactionManager(@Qualifier("shardDataSource") DataSource dataSource) {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }

    public static TableRuleConfiguration getTableRuleConfiguration(String tableName, String shardingColumn, String actualDataNodes) {
        return getTableRuleConfiguration(tableName, shardingColumn, actualDataNodes, 2, 2);
    }

    public static TableRuleConfiguration getTableRuleConfiguration(String tableName,String shardingColumn,String actualDataNodes,Integer databaseCount,Integer tableCount){

        TableRuleConfiguration  tableRuleConfiguration = new TableRuleConfiguration(tableName, actualDataNodes);
        tableRuleConfiguration.setDatabaseShardingStrategyConfig(new StandardShardingStrategyConfiguration(shardingColumn, (PreciseShardingAlgorithm<Long>) (availableTargetNames, shardingValue) -> {
            String shard = "_" + (shardingValue.getValue() % (databaseCount*10) / 10);
            return availableTargetNames.stream().filter(name -> name.endsWith(shard)).findFirst().orElseThrow(IllegalArgumentException::new);
        }));
        tableRuleConfiguration.setTableShardingStrategyConfig(new StandardShardingStrategyConfiguration(shardingColumn, (PreciseShardingAlgorithm<Long>) (availableTargetNames, shardingValue) -> {
            String shard = "_" + (shardingValue.getValue() % tableCount);
            return availableTargetNames.stream().filter(name -> name.endsWith(shard)).findFirst().orElseThrow(IllegalArgumentException::new);
        }));


        return tableRuleConfiguration;
    }


    public static void main(String[] args) {
        Integer databaseCount = 5;
        Integer tableCount = 10;
        Long id = 2595280942797427L;
        String shard = "_" + (id % (databaseCount * 10) / 10 + 1);
        String shard2 = "_" + (id % tableCount + 1);
        System.out.println(shard + "====" +shard2);

//        String shard = "_" + (int) (Math.ceil( 557309728 % 50 / 10 + 1));
//        String shard2 = "_" + (557309728 % 10);
//        System.out.println(shard + "====" +shard2);

//        String s = "12321321";
//        System.out.println(StringUtils.isNumeric(s));
    }
}
```



