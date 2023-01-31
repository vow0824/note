###### docker 安装elasticsearch

存储和检索数据

```shell
sudo docker pull elasticsearch:7.4.2
```

可视化检索数据

```shell
sudo docker pull kibana:7.4.2  
```

elasticsearch

```shell
sudo mkdir -p /mydata/elasticsearch/config
```

```shell
sudo mkdir -p /mydata/elasticsearch/data
```

```shell
sudo echo "http.host: 0.0.0.0">>/mydata/elasticsearch/config/elasticsearch.yml
```

修改文件夹权限，保证权限问题，否则启动报错

```shell
sudo chmod -R 777 /mydata/elasticsearch/
```

```shell
sudo docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2
```

特别注意：

-e ES_JAVA_OPTS="-Xms64m -Xmx512m" 测试环境下，设置ES的初始值和最大容量，否则导致过大启动不了ES

设置elasticsearch容器启动自启动

```shell
sudo docker update elasticsearch --restart=always
```

给elasticsearch.yml添加如下配置，允许所有请求和跨域请求

```tex
http.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```

kibana

```shell
sudo docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.43.219:9200 -p 5601:5601 \
-d kibana:7.4.2
```

```shell
sudo docker update kibana --restart=always
```

```shell
sudo docker run --name kibana -e ELASTICSEARCH_HOSTS=[http://192.168.43.97:9200](http://192.168.43.219:9200/) -p 5601:5601 \
-d kibana:7.4.2
```

