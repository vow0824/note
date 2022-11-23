centos8切换yum源，在未安装wget命令的情况下

进入root，切换至yum.repos.d目录

```shel
cd /etc/yum.repos.d/
```

创建新文件夹并将源文件备份为repo.bak
```shell
mkdir backup && mv *repo backup/
```

下载国内yum源文件
```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

更新下载yum源地址
```shell
sed -i -e"s|mirrors.cloud.aliyuncs.com|mirrors.aliyun.com|g " /etc/yum.repos.d/CentOS-*
sed -i -e "s|releasever|releasever-stream|g" /etc/yum.repos.d/CentOS-*
```

生成缓存
```shell
yum clean all && yum makecache
```

