---
layout: post
title:  Docker入坑 
date: 2017-12-17
tag: others
---

## Docker   

**入坑docker嘿嘿**  


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
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /ect/apt/sources.list \
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


**持续入坑**  

<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
````

````

````