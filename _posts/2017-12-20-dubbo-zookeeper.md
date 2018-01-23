---
layout: post
title:  dubbo-zookeeper 
date: 2017-12-20
tag: others
---

<br>

**之前在本地搭建过Dubbo与Zookeeper、Spring的整合,既然有了docker就更方便了,这里记下笔记**  

````
#运行 zookeeper:用的是官方zookeeper镜像
docker run -d --name zookeeper -p 2181:2181 zookeeper:latest

#运行 dubboadmin:在dockerhub上找的一个dubboadmin镜像
docker run -d --name dubboadmin -p 8082:8080 --link zookeeper:zk lemonguge/dubbo-admin:latest

访问ip:8082就好啦  root/root  guest/guest

````

dubbo与spring的demo实现了生成者消费者和公共API，代码在github: [dubboDemo](https://github.com/tomsun28/dubboDemo)  


<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)

