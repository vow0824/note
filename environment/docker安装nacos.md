获取镜像

```shell
docker pull nacos/nacos-server
```

创建文件夹

```shell
mkdir -p /mydata/nacos/logs
mkdir -p /mydata/nacos/init.d
touch /mydata/nacos/init.d/custom.properties
vi /mydata/nacos/init.d/custom.properties
```

custom.properties

```properties
management.endpoints.web.exposure.include=*
```

启动nacos

```shell
docker run \
--name nacos -d \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-e SPRING_DATASOURCE_PLATFORM=mysql  \
-e MYSQL_SERVICE_HOST=192.168.119.16 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos_config \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=root \
-v /mydata/nacos/logs:/home/nacos/logs \
-v /mydata/nacos/init.d/custom.properties:/etc/nacos/init.d/custom.properties \
-v /mydata/nacos/data:/home/nacos/data \
nacos/nacos-server
```

```markdown
docker 启动容器
docker run \
 
容器名称叫nacos -d后台运行
--name nacos -d \
 
nacos默认端口8848 映射到外部端口8848
-p 8848:8848 \
 
naocs 应该是2.0版本以后就需要一下的两个端口 所以也需要开放
-p 9848:9848 
-p 9849:9849 
--privileged=true \
 
docker重启时 nacos也一并重启
--restart=always \
 
-e 配置 启动参数
配置 jvm
-e JVM_XMS=256m 
-e JVM_XMX=256m \
 
单机模式
-e MODE=standalone 
-e PREFER_HOST_MODE=hostname \
 
数据库是mysql 配置持久化 不使用nacos自带的数据库
-e SPRING_DATASOURCE_PLATFORM=mysql \
 
写自己的数据库地址
-e MYSQL_SERVICE_HOST=###### \
 
数据库端口号
-e MYSQL_SERVICE_PORT=3306 \
 
mysql的数据库名称
-e MYSQL_SERVICE_DB_NAME=nacos \
 
mysql的账号密码
-e MYSQL_SERVICE_USER=root 
-e MYSQL_SERVICE_PASSWORD=root \
 
-v 映射docker内部的文件到docker外部 我这里将nacos的日志 数据 以及配置文件 映射出来
映射日志
-v /root/apply/docker/apply/nacos/logs:/home/nacos/logs \
 
映射配置文件 (应该没用了 因为前面已经配置参数了)
-v /root/apply/docker/apply/nacos/init.d/custom.properties:/etc/nacos/init.d/custom.properties \
 
映射nacos的本地数据 也没啥用因为使用了mysql
-v /root/apply/docker/apply/nacos/data:/home/nacos/data \
 
启动镜像名称
nacos/nacos-server
```

