docker 安装 mysql

```shell
sudo docker pull mysql:5.7
```



```shell
sudo docker run -p 3306:3306 --name mysql \
  -v /mydata/mysql/log:/var/log/mysql \
  -v /mydata/mysql/data:/var/lib/mysql \
  -v /mydata/mysql/conf:/etc/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -d mysql:5.7
```



-p 3306:3306 --name mysql ：将容器的3306端口映射到主机的3306端口

-v /mydata/mysql/conf:/etc/mysql ：将配置文件夹挂载到主机

-v /mydata/mysql/log:/var/log/mysql ：将日志文件挂载到主机

-v /mydata/mysql/log:/var/log/mysql ：将配置文件夹挂载到主机

-e MYSQL_ROOT_PASSWORD=root ：初始化用户密码



修改配置文件

```java
sudo vi /mydata/mysql/conf/my.cnf
```



mysql 配置

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
```



设置mysql 容器启动时自启动

```shell
sudo docker update mysql --restart=always
```



重启mysql

```shell
sudo docker restart mysql
```

