---
layout: post
title:  Docker学习 
date: 2017-12-17
tag: others
---

## Docker   


### 入坑docker嘿嘿  

**ubantu安装docker**  

[官方安装教程](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#supported-storage-drivers)

**docker常用命令**  

````
#用Dockerfile构建镜像
docker build -t tomsun28/ubantu:6.0 .

#通过docker镜像生成docker容器
#-it是交互式模式(对应-d是后台启动)
#-p 用本机的8080端口映射容器的80端口
docker run -it -p 8080:80 --name containerName tomsun28/ubantu:6.0


#进入正在运行的容器内
docker exec -it containerName /bin/bash


docker images 查看镜像
docker ps -a  查看所有容器状态
docker rm 删除容器
docker rmi 删除镜像

systemctl restart docker 重启docker

````

**docker 构建mariadb**  

````
#用官方镜像启动mariadb,将存放数据库信息的文件夹/var/lib/mysql映射到本地
docker run -d -p 3306:3306 --name mariadb -e TIMEZONE=Asia/Shanghai -v /mnt/dockerWorkspace/mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=admin  mariadb

#进入mariadb容器
docker exec -it mariadb /bin/bash

#登录数据库
mysql -uroot -padmin

````


**Dockerfile----基于Ubuntu基础镜像生成的Tomcat中间件镜像**  

````

#VERSION 1.0.0
#基础镜像为ubuntu
FROM ubuntu

#签名
MAINTAINER tomsun28 "tomsun28@outlook.com"

#RUN一次就会构建一层镜像,层次多会产生不必要的臃肿
#更新源,安装ssh server
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list \
    && apt-get update \
	&& apt-get install -y openssh-server \
	&& mkdir -p /var/run/sshd \
#设置root ssh远程登录密码
    && echo "root:root" | chpasswd \
#安装ppa源,一次性安装vim,wget,curl,java,tomcat
    && apt-get install python-software-properties \
	&& add-apt-repository ppa:webupd8team/java \
	&& apt-get update \
	&& apt-get install -y vim wget curl oracle-java8-installer tomcat8 \
#设置JAVA_HOME环境变量
    && update-alternatives --dispaly java \
	&& echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /ect/environment \
	&& echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /ect/default/tomcat8


#开放SSH的22端口
EXPOSE 22
#开放tomcat的8080端口
EXPOSE 8080

#设置tomcat8初始化运行,ssh终端服务器作为后台运行
ENTRYOINT service tomcat8 start && usr/sbin/sshd -D


````

### 使用Docker持续集成与自动化部署

**自己用docker搭了一套项目开发的持续集成部署，在这个记录下流程万一又给忘了**  

````
#构建Docker私有仓库
docker run -d --restart=always --name registry \
-v /mnt/dockerWorkspace/registry:/tmp/registry -p 5000:5000 registry
#向仓库推送镜像
docker push 182.61.59.000:5000/tomcat:1.0
#向仓库拉取镜像
docker pull 182.61.59.000:5000/tomcat:1.0
#server gave HTTP response to HTTPS client问题
touch /etc/docker/daemon.json
echo "{ "insecure-registries":["182.61.59.000:5000"] }" >> /etc/docker/daemon.json

````

````
#构建Jenkins
docker run -d -p 8080:8080 --name jenkins --restart=always \
-v /mnt/dockerWorkspace/jenkins_home:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock  jayqqaa12/jenkins
#参数/mnt/dockerWorkspace/jenkins_home:/var/jenkins_home将目录映射到本地磁盘
#参数/var/run/docker.sock:/var/run/docker.sock映射本地docker,这样就能在jenkins容器使用docker
默认用户密码 admin/admin


````

**Dockerfile     tomcat项目所对应的tomcat镜像构建Dockerfile,将此Dockerfile放在当前项目下,示例项目名为WebHelloWorld**  

````
#VERSION 1.0.0
#基础镜像为tomcat
FROM tomcat:8-jre8

#签名
MAINTAINER tomsun28 "tomsun28@outlook.com"


#加入WAR包到tomcat下
RUN rm -rf /usr/local/tomcat/webapps
ADD ./target/WebHelloWorld.war /usr/local/tomcat/webapps/WebHelloWorld.war
````

**Jenkins细节配置**

````
配置->项目名称：最好为github上代码的项目名称,必须小写
配置->源码管理->Git：URL为github上的项目clone url,下面默认master分支
配置->触发远程构建选上->身份证令牌：eg:hahaha

用户->设置->查看API Token:    userID=xxxx,apiToken=xxxx
这两个github配置会用到
````




**Jenkins配置构建步骤shell脚本**  

````

#!/bin/bash

#build in jenkins sh

#docker docker hub仓库地址,之后把生成的镜像上传到  registry or docker hub
REGISTRY_URL=182.61.59.218:5000
#docker login --username tomsun28 --password usthecom123

#根据时间生成版本号
TAG=$REGISTRY_URL/$JOB_NAME:`date +%y%m%d-%H-%M`

#使用maven 镜像进行编译 打包出 war 文件
docker run --rm --name mvn -v /mnt/dockerWorkspace/maven:/root/.m2 \
-v /mnt/dockerWorkspace/jenkins_home/workspace/$JOB_NAME:/usr/src/mvn -w /usr/src/mvn/ \
maven:3.3.3-jdk-8 mvn clean install -Dmaven.test.skip=true

#使用放在项目下面的Dockerfile打包生成镜像
docker build -t $TAG $WORKSPACE/.

docker push $TAG
docker rmi $TAG

#判断之前运行的容器是否还在，在就删除
if docker ps -a | grep -i $JOB_NAME;then
docker rm -f $JOB_NAME
fi

#用最新版本的镜像运行容器

docker run -d -p 80:8080 --name $JOB_NAME $TAG

````

**github配置**

````
对应项目->settings->Webhooks->add webhook:
Payload URL = http://userID:apiToken@182.61.59.000:8080/job/webhelloworld/build?token=hahaha
userID,apiToken为上面Jenkins的userID,apiToken的值
182.61.59.000:8080为Jenkins的IP:端口
WebHelloWorld为Jenkins的项目名
hahaha为上面Jenkins设置的身份令牌

````



<br>
**持续入坑**  

<br>
<br>
<br>

*参考来自*[使用Docker构建持续集成与自动部署的Docker集群](http://blog.csdn.net/java_dyq/article/details/51997024)  
*转载请注明* [from tomsun28](http://usthe.com)  

