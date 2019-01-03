---
layout: post
title:  基于kubernetes+docker+jenkins的DevOps实践 
date: 2019-01-02
tag: CICD
---

<br>
<br>

## 基于kubernetes+docker+jenkins的DevOps实践     

之前自己的项目开发就搭了个cicd的环境，那时候是在本就小的可怜的服务器上搭了一套    
```jenkins + docker registry + docker```   
见之前的笔记 [docker学习](http://usthe.com/2017/12/docker_learn/)下面   
总的差不多这样:  
![image1](/images/posts/cicd/image1.PNG)  

<br>
之后对```kubernetes```的接触后，就在之前的基础上加入```kubernetes```,其实也就是在服务器拉取镜像```docker run```的时候改变为通知```kubernetes```的```apiServer```对提前配置好的项目配置文件```xx.yaml```进行更新```kubectl appply -f xx.yaml```，它会对配置里的镜像拉取在多个```pod```里运行，当然还需要对应的```service```，如果需要暴露给外部还可以添个```ingress```。  

<br>
一个小服务器加本地一个闲置从机撑进去这么多东西很显然爆了，于是把```jenkins , docker registry```拆出来，用上了公共的ali云服务```CodePipeline,容器镜像服务```。  
这里记录一下。  

- - -

### docker搭建  

[ubuntu安装docker官方教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)  


### kubernetes搭建  

之前写的[kubernetes学习](https://segmentfault.com/a/1190000017488220)下面有  


### 使用ali云CodePipeline替代jenkins创建任务  

````
配置->项目名称：最好为github上代码的demo项目名称,这里以bootshiro为例
配置->源码管理->Git：URL为github上的项目clone url,下面默认master分支
配置->构建触发器->填写代码分支：eg:master 点击生成触发器地址留下备用(github webhook配置会用到)
````
![image1](/images/posts/cicd/image2.PNG)  
![image1](/images/posts/cicd/image3.PNG)  

````
配置->构建项目类型可选maven项目 node python等(按自己需求改编译打包册测试脚本)
eg: maven项目 编译打包: mvn package -B -DskipTests 用例测试: mvn test
````
![image1](/images/posts/cicd/image4.PNG)  

````
配置->镜像构建和发布: 这里使用ali云的免费docker镜像仓库
镜像版本号最好用jenkins环境变量标记,registry地址证书等就是自己开通的ali云registry地址和账户,docker路径是相对于当前代码仓库的Dcokerfile文件路径,用这个Dockefile文件来生成镜像。
eg: bootshiro的Dockefile
#VERSION 1.1.0
#基础镜像为openjdk:12-alpine

FROM openjdk:12-alpine

#签名
MAINTAINER tomsun28 "tomsun28@outlook.com"


RUN rm -rf /opt/running/bootshiro*
ADD ./target/bootshiro.jar /opt/running/bootshiro.jar

EXPOSE 8080
WORKDIR /opt/running/

CMD ["java", "-jar", "bootshiro.jar","--spring.profiles.active=prod"]
````
![image1](/images/posts/cicd/image5.PNG)  

````
配置->部署Kubernetes(新): 这里配置对搭建好的k8s环境的apiServer连接,之后好使用apiServer对kubernetes操作
认证方式：选择认证证书
API服务器地址：为apiServer通讯地址
证书：使用docker授权模式,客户端Key(key.pem)和客户端证书(cert.pem)在/etc/kubernetes/admin.conf,服务端CA证书(ca.pem)在/etc/kubernetes/pki/ca.crt
部署配置文件：为k8s部署这个项目的配置文件位置，也是以当前项目代码仓库为相对路径,eg :bootshiro.yaml

# ----------------------bootshiro--------------------- #

# ------bootshiro deployment------ #
kind: Deployment
apiVersion: apps/v1beta2
metadata:
 name: bootshiro-deployment
 labels: 
  app: bootshiro
spec:
 replicas: 1
 selector:
  matchLabels:
   app: bootshiro
 template:
  metadata:
   labels:
    app: bootshiro
  spec:
   containers:
   - name: nginx
     image: registry.cn-hangzhou.aliyuncs.com/tomsun28/bootshiro:${BUILD_NUMBER}
     ports:
     - containerPort: 8080

---

# -------nginx-service--------- #
apiVersion: v1
kind: Service
metadata:
 name: bootshiro-service
spec:
# type: NodePort
 ports:
 - name: server
   port: 8080
   targetPort: 8080
 selector:
  app: bootshiro

# !----------------------bootshiro--------------------- #

这里配置部署文件创建了一个pod实例，创建了与其想对应的service在集群内部暴露服务。
如果部署的应用需要对集群外提供服务，这里还要创建对应的暴露服务的方式，如ingress, nodeport等
````
![image1](/images/posts/cicd/image6.PNG)  

<br>
- - -

到此cicd就差不多了，我们开发代码push到github仓库上，跟着DevOps流程走，最后项目就会自己运行到kubernetes集群里面了，pod挂了或者从机挂了，k8s会重新启保证设定数量的pod。

### 使用ingress对集群外暴露服务  

这里使用的是```traefik-ingress```,在kubernetes中部署traefik有[官方部署手册](https://docs.traefik.io/user-guide/kubernetes/)，基本按着走一遍就能部署上去了。  

整合部署的traefik.yaml:  

````
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: PathPrefixStrip

spec:
  rules:
  - host: tom.usthe.com
    http:
      paths:
      - path: /ingress
        backend:
          serviceName: traefik-web-ui
          servicePort: web
````
使用```traefik来暴露service```:
````
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: chess
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: PathPrefixStrip
spec:
  rules:
  - host: tom.usthe.com
    http:
      paths:
      - path: /usthe
        backend:
          serviceName: usthe-service
          servicePort: http
      - path: /nginx
        backend:
          serviceName: nginx
          servicePort: http
````
![image1](/images/posts/cicd/image7.PNG)  






<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
