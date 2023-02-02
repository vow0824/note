docker 安装redis

```shell
sudo docker pull redis
```



不指定版本号，默认下载最新版

创建实例并启动

```shell
sudo mkdir -p /mydata/redis/conf

sudo touch /mydata/redis/conf/redis.conf
```



不先创建出redis.conf文件，等会挂载时可能会指定为文件夹导致挂载失败

```shell
sudo docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```



设置redis 容器启动时自启动

```shell
sudo docker update redis --restart=always
```



使用redis镜像实行redis-cli命令连接

```shell
sudo docker exec -it redis redis-cli
```



在redis.conf里面加入

```properties
appendonly yes
```

确保redis的持久化，否则redis重启，数据清空