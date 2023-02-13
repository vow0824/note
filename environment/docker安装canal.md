###### docker安装canal

```shell
docker pull canal/canal-server:v1.1.5

mkdir -p /mydata/canal/conf

# 创建一个容器
docker run --name canal -d canal/canal-server:v1.1.5

# 复制容器中的配置文件到本地
docker cp canal:/home/admin/canal-server/conf/canal.properties /mydata/canal
docker cp canal:/home/admin/canal-server/conf/example/instance.properties /mydata/canal
```

删除之前的实例，重新启动canal实例

```shell
docker run --name canal -p 11111:11111 \
-v /mydata/canal/conf/instance.properties:/home/admin/canal-server/conf/example/instance.properties \
-v /mydata/canal/conf/canal.properties:/home/admin/canal-server/conf/canal.properties \
-d canal/canal-server:v1.1.5
```

```shell
sudo docker update canal --restart=always
```

