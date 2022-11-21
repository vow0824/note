###### Linux配置java环境

检查虚拟机是否已安装java

```
rpm -qa|grep java
```

检索安装源

```
yum list java*
```

安装

```
yum install java-1.8.0-openjdk.x86_64
```

通过yum安装的好处就是已经自动帮我们设置好环境变量