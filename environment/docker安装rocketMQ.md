###### docker安装rocketMQ

搜索镜像

```shell
docker search rocketmq
```

获取镜像

```shell
docker pull rocketmqinc/rocketmq
```



启动nameserver

创建本地数据目录

```shell
mkdir -p /mydata/rocketmq/nameserver/logs /mydata/rocketmq/nameserver/store
```

```shell
docker run -d \
--restart=always \
--name rmqnamesrv \
-p 9876:9876 \
-v /docker/rocketmq/data/namesrv/logs:/root/logs \
-v /docker/rocketmq/data/namesrv/store:/root/store \
-e "MAX_POSSIBLE_HEAP=100000000" \
rocketmqinc/rocketmq \
sh mqnamesrv
```



启动broker

创建brocker数据存储路径

```shell
mkdir -p  /mydata/rocketmq/data/broker/logs   /mydata/rocketmq/data/broker/store /mydata/rocketmq/conf
```



创建broker配置文件

```shell
vi /mydata/rocketmq/conf/broker.conf
```

```properties
# 所属集群名称，如果节点较多可以配置多个
brokerClusterName = DefaultCluster
#broker名称，master和slave使用相同的名称，表明他们的主从关系
brokerName = broker-a
#0表示Master，大于0表示不同的slave
brokerId = 0
#表示几点做消息删除动作，默认是凌晨4点
deleteWhen = 04
#在磁盘上保留消息的时长，单位是小时
fileReservedTime = 48
#有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
brokerRole = ASYNC_MASTER
#刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
flushDiskType = ASYNC_FLUSH
# 设置broker节点所在服务器的ip地址
brokerIP1 = 192.168.28.16
# 磁盘使用达到95%之后,生产者再写入消息会报错 CODE: 14 DESC: service not available now, maybe disk full
diskMaxUsedSpaceRatio=95

```

启动broker

```shell
docker run -d  \
--restart=always \
--name rmqbroker \
--link rmqnamesrv:namesrv \
-p 10911:10911 \
-p 10909:10909 \
-v  /mydata/rocketmq/data/broker/logs:/root/logs \
-v  /mydata/rocketmq/data/broker/store:/root/store \
-v /mydata/rocketmq/conf/broker.conf:/opt/rocketmq-latest/conf/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=200000000" \
rocketmqinc/rocketmq \
sh mqbroker -c /opt/rocketmq-latest/conf/broker.conf
```



创建rockermq-console服务

```shell
docker pull pangliang/rocketmq-console-ng
```

启动服务

```shell
docker run -d \
--restart=always \
--name rmqadmin \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.28.16:9876 \
-Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9999:8080 \
pangliang/rocketmq-console-ng
```



关闭防火墙

```shell
systemctl stop firewalld.service
```

开放指定端口

```shell
firewall-cmd --permanent --zone=public --add-port=9876/tcp
firewall-cmd --permanent --zone=public --add-port=10911/tcp
# 立即生效
firewall-cmd --reload
```

