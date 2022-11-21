###### Linux安装maven

配置yum源
 ```
 wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
 ```

```
sed -i s/\$releasever/``6``/g /etc/yum.repos.d/epel-apache-maven.repo
```

安装

```
yum install -y maven
```

配置阿里云镜像

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

