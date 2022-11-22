###### Linux安装docker



1、参考docker

Older versions of Docker were called docker or docker-engine. If these are installed, uninstall them, along with associated dependencies.

```shell
$ sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
```



Install the yum-utils package (which provides the yum-config-manager utility) and set up the **stable** repository.

```shell
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```



Install the *latest version* of Docker Engine and containerd, or go to the next step to install a specific version:

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```



解决与podman依赖的冲突------卸载podman：yum remove podman

Last metadata expiration check: 0:00:02 ago on Fri 25 Sep 2020 02:39:18 PM CST.

Error:

Problem 1: problem with installed package podman-1.6.4-10.module_el8.2.0+305+5e                                                       198a41.x86_64

 \- package podman-1.6.4-10.module_el8.2.0+305+5e198a41.x86_64 requires runc >=                                                       1.0.0-57, but none of the providers can be installed

 \- package containerd.io-1.3.7-3.1.el8.x86_64 conflicts with runc provided by r                                                       unc-1.0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- package containerd.io-1.3.7-3.1.el8.x86_64 obsoletes runc provided by runc-1                                                       .0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- conflicting requests

 \- package runc-1.0.0-64.rc10.module_el8.2.0+304+65a3c2ac.x86_64 is filtered ou                                                       t by modular filtering

Problem 2: problem with installed package buildah-1.11.6-7.module_el8.2.0+305+5                                                       e198a41.x86_64

 \- package buildah-1.11.6-7.module_el8.2.0+305+5e198a41.x86_64 requires runc >=                                                       1.0.0-26, but none of the providers can be installed

 \- package containerd.io-1.3.7-3.1.el8.x86_64 conflicts with runc provided by r                                                       unc-1.0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- package containerd.io-1.3.7-3.1.el8.x86_64 obsoletes runc provided by runc-1                                                       .0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- package docker-ce-3:19.03.13-3.el8.x86_64 requires containerd.io >= 1.2.2-3,                                                       but none of the providers can be installed

 \- conflicting requests

 \- package runc-1.0.0-56.rc5.dev.git2abd837.module_el8.2.0+303+1105185b.x86_64                                                       is filtered out by modular filtering

 \- package runc-1.0.0-64.rc10.module_el8.2.0+304+65a3c2ac.x86_64 is filtered ou                                                       t by modular filtering

(try to add '--allowerasing' to command line to replace conflicting packages or                                                       '--skip-broken' to skip uninstallable packages or '--nobest' to use not only bes                                                       t candidate packages)

解决其他依赖冲突或依赖缺失-----   sudo yum install docker-ce docker-ce-cli containerd.io --allowerasing

Error:

Problem: problem with installed package buildah-1.11.6-7.module_el8.2.0+305+5e198a41.x86_64

 \- package buildah-1.11.6-7.module_el8.2.0+305+5e198a41.x86_64 requires runc >= 1.0.0-26, but none of the providers can be installed

 \- package containerd.io-1.3.7-3.1.el8.x86_64 conflicts with runc provided by runc-1.0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- package containerd.io-1.3.7-3.1.el8.x86_64 obsoletes runc provided by runc-1.0.0-65.rc10.module_el8.2.0+305+5e198a41.x86_64

 \- conflicting requests

 \- package runc-1.0.0-56.rc5.dev.git2abd837.module_el8.2.0+303+1105185b.x86_64 is filtered out by modular filtering

 \- package runc-1.0.0-64.rc10.module_el8.2.0+304+65a3c2ac.x86_64 is filtered out by modular filtering

(try to add '--allowerasing' to command line to replace conflicting packages or '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)



设置docker开机自启动
sudo systemctl enable docker

阿里云配置镜像加速器



## **2. 配置镜像加速器**

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://i1h74lam.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



常用命令

docker pull mysql:5.7

docker run ...

docker ps

docker images

docker exec





docker要删除镜像，先要删除依赖它的容器

1. 删除容器

docker ps #查看正在运行的容器

docker ps -a #查看所有容器

docker rm container_id #删除容器

2. 删除镜像

docker images //查看镜像

docker rmi image_id

2.1 删除其他镜像

删除 null image

sudo docker rmi $(docker images -f "dangling=true" -q) #删除所有镜像

删掉容器

docker stop $(docker ps -qa)
docker rm $(docker ps -qa)

删除镜像

docker rmi --force $(docker images -q)

删除名称中包含某个字符串的镜像

例如删除包含“some”的镜像

docker rmi --force $(docker images | grep some | awk '{print $3}')



docker查看日志的几个方式：

（1）docker logs --tail=1000 容器名称 （查看容器前多少行的日志）（推荐）

（2）docker 容器启动后,可以进入以下位置查看日志（/var/lib/docker/containers/容器ID/容器ID-json.log）（进入容器内部查看日志）

（3）#查看compose所有容器的运行日志

docker-compose -f docker-compose-app.yml logs -f

（4）#查看compose下某个容器的运行日志

docker-compose -f docker-compose-app. yml logs -f<服务名>

（5）# 也可以把compose的容器日志输出到日志文件里去，然后用tail -f随时查看

docker-compose -f docker-compose-app. yml logs -f >> myDockerCompose.log &
