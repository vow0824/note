###### canal同步mysql数据配置

设置用户

```sql
create user canal identified by 'canal';
grant select,replication slave, replication client on *.* to 'canal'@'%';
flush privileges;

# 查看bin-log是否开启 on: 开启 off: 关闭
show variables like 'log_bin'; 
```

mysql的配置文件添加binlog配置

```properties
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# start binlog
log-bin=mysql-bin
binlog-format=ROW
server_id=1
```

查看mysql的binlog日志情况

```sql
# 查看binlog文件列表
show binary logs;
# 查看当前正在写入的binlog文件
show master status;

# 查看指定binlog文件的内容
show binlog events [in 'log_name'] [FROM pos] [limit [offset,] row_count]

```

配置canal

查看mysql的ip

```shell
docker inspect mysql  # 这里假设此处ip是 177.17.0.1
```

修改canal.properties配置文件

```properties
# 默认端口 11111
# 默认输出model为tcp, 这里根据使用的mq类型进行修改
# tcp, kafka, RocketMQ
canal.serverMode = tcp

#################################################
######### destinations ############# 
#################################################
# canal可以有多个instance,每个实例有独立的配置文件，默认只 有一个example实例。
# 如果需要处理多个mysql数据的话，可以复制出多个example,对其重新命名，
# 命令和配置文件中指定的名称一致。然后修改canal.properties 中的 canal.destinations
# canal.destinations=实例 1，实例 2，实例 3
canal.destinations = example

```

修改instance.properties配置文件

```properties
# 不能和mysql重复
canal.instance.mysql.slaveId=2
# 使用mysql的虚拟ip和端口
canal.instance.master.address=177.17.0.1:3306
# 使用已创建的canal用户
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
# canal.instance.defaultDatabaseName =test

# 问题：（原本这样的，值同步test库，此处没能解决，单据指定数据库同步配置）
# canal.instance.filter.regex=.*\\..*
# canal.instance.defaultDatabaseName =test

# 注掉上面，然后添加，同步所有的库。
# .\*\\\\..\*:  表示匹配所有的库所有的表
canal.instance.filter.regex =.\*\\\\..\*

# 目的地，可以认识一个消息队列，不需要更改。
canal.mq.topic=example
```



创建canal_tsdb数据库并创建记录表结构变化记录表【可选操作】

```sql
CREATE TABLE IF NOT EXISTS `meta_snapshot` (
                                               `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
                                               `gmt_create` datetime NOT NULL COMMENT '创建时间',
                                               `gmt_modified` datetime NOT NULL COMMENT '修改时间',
                                               `destination` varchar(128) DEFAULT NULL COMMENT '通道名称',
                                               `binlog_file` varchar(64) DEFAULT NULL COMMENT 'binlog文件名',
                                               `binlog_offest` bigint(20) DEFAULT NULL COMMENT 'binlog偏移量',
                                               `binlog_master_id` varchar(64) DEFAULT NULL COMMENT 'binlog节点id',
                                               `binlog_timestamp` bigint(20) DEFAULT NULL COMMENT 'binlog应用的时间戳',
                                               `data` longtext DEFAULT NULL COMMENT '表结构数据',
                                               `extra` text DEFAULT NULL COMMENT '额外的扩展信息',
                                               PRIMARY KEY (`id`),
                                               UNIQUE KEY binlog_file_offest(`destination`,`binlog_master_id`,`binlog_file`,`binlog_offest`),
                                               KEY `destination` (`destination`),
                                               KEY `destination_timestamp` (`destination`,`binlog_timestamp`),
                                               KEY `gmt_modified` (`gmt_modified`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='表结构记录表快照表';

CREATE TABLE IF NOT EXISTS `meta_history` (
                                              `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
                                              `gmt_create` datetime NOT NULL COMMENT '创建时间',
                                              `gmt_modified` datetime NOT NULL COMMENT '修改时间',
                                              `destination` varchar(128) DEFAULT NULL COMMENT '通道名称',
                                              `binlog_file` varchar(64) DEFAULT NULL COMMENT 'binlog文件名',
                                              `binlog_offest` bigint(20) DEFAULT NULL COMMENT 'binlog偏移量',
                                              `binlog_master_id` varchar(64) DEFAULT NULL COMMENT 'binlog节点id',
                                              `binlog_timestamp` bigint(20) DEFAULT NULL COMMENT 'binlog应用的时间戳',
                                              `use_schema` varchar(1024) DEFAULT NULL COMMENT '执行sql时对应的schema',
                                              `sql_schema` varchar(1024) DEFAULT NULL COMMENT '对应的schema',
                                              `sql_table` varchar(1024) DEFAULT NULL COMMENT '对应的table',
                                              `sql_text` longtext DEFAULT NULL COMMENT '执行的sql',
                                              `sql_type` varchar(256) DEFAULT NULL COMMENT 'sql类型',
                                              `extra` text DEFAULT NULL COMMENT '额外的扩展信息',
                                              PRIMARY KEY (`id`),
                                              UNIQUE KEY binlog_file_offest(`destination`,`binlog_master_id`,`binlog_file`,`binlog_offest`),
                                              KEY `destination` (`destination`),
                                              KEY `destination_timestamp` (`destination`,`binlog_timestamp`),
                                              KEY `gmt_modified` (`gmt_modified`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='表结构变化明细表';
```

