---
layout: post
title:  kubernetes learn
date: 2018-07-24
tag: 分布式
---

###  kubernetes  

#### kubernetes 知识点  

**1. 核心组件**  

- etcd 保存整个集群的状态信息，感觉相当于k8s的数据库  
- apiserver 提供对k8s资源操作的唯一入口，并提供认证授权，访问控制，API注册与发现等机制  
- controller manager 负责维护集群的状态，eg:故障检测，自动扩展pod，滚动更新等  
- scheduler 负责对资源的调度，按着预定的调度策略将pod调度到相应的集群上  
- kubelet 负责维护容器的生命周期，相当于在node上的agent，负责管理pods和它们上面的容器，images镜像、volumes等  
- kube-proxy 负责为service提供集群内部的服务发现和负载均衡  

**2. kubernetes 常用命令**  

- 查看集群信息  
```
kubectl cluster-info
```

- 在集群中运行一个应用程序  
```
kubectl run nginx-test  --replicas=3 --labels='app=nginx' --image=nginx:latest --port=80  
#使用kubectl run命令启动一个pod，自定义名称为nginx-test，启动了3个pod副本，并给pod打上标签app=nginx，这个pod拉取docker镜像nginx:latest，开放端口80
```

- 查看集群中所有pod  
```
kubectl get po
kubectl get pod
kubectl get pods
```

- 根据标签label查看集群中pod  
```
kubectl get pods -l app
kubectl get pods -l app=nginx
```

- 查看标签为app=nginx的pod在集群中具体分配在哪个节点和pod的ip  
```
kubectl get pods -l app=nginx -o wide
```

- 查看pod的详细信息  
```
kubectl describe pods <podname>
```

- 查看集群中的deployment(其他命令与pod类似)  
```
kubectl get deploy
```

- 查看集群中的replica set(其他命令与pod类似)  
```
kubectl get replicaset
kubectl get rs
```

- 创建一个service，集群中的资源通过service与外界交互  
```
kubectl expose deploy nginx-test --port=8080 --target-port=80 --name=nginx-service
#k8s集群通过deploy来管理，导出名为nginx-test的deploy，为其创建名为nginx-service的服务开放给外界，使外界能通过nginx-service来和nginx-test交互，外部端口为8080,内部端口为80
```

- 查看集群中的服务(其他命令与pod类似)  
```
kubectl get svc
```

- 查看pod中容器的日志  
```
kubectl log <podname>    #查看指定pod内容器的日志  
kubectl log -l app=nginx #查看标签lable为app=nginx下的pod的容器日志
```

- pod的副本的扩容和缩容  
```
kubectl scale deploy nginx-test --replicas10
#通过kubectl scale将名为nginx-test的deploy重新定义有10个副本pod
```

- 查看pod副本扩容缩容的实时进度  
```
kubectl rollout status deploy nginx-test
```

- 删除资源  
**pod和rs不能直接被删除,其被deploy控制,即使删除了某一pod,也会创建新的pod来对应配置pod副本数量,要想删除pod,只能用删除其deploy来删除,或者变更pod副本配置缩容(如上)**

```
kubectl delete deploy nginx-test  #删除部署的deploy(删除其对于的pod和rs)
kubectl delete svc nginx-service  #删除创建的service

```


**3. 应用创建部署yaml文件**  

**tomsun28之后的k8s应用部署修改，都确定使用apply形式部署更新，使用git版本控制创建资源，好处多多**  

```
kubectl apply -f nginx.yaml    ##更新式创建资源，如果不存在此资源则创建，如存在改动则调整资源(推荐)
kubectl delete -f nginx.yaml   #资源(pod,deployment,service,replicaset...)删除销毁
```

- kubernetes部署nginx集群  

```nginx.yaml :```

```
# ----------------------nginx--------------------- #

# ------nginx deployment------ #
kind: Deployment
apiVersion: apps/v1beta2
metadata:
 name: nginx-deployment
 labels: 
  app: nginx
spec:
 replicas: 3
 selector:
  matchLabels:
   app: nginx
 template:
  metadata:
   labels:
    app: nginx
  spec:
   containers:
   - name: nginx
     image: 192.167.2.144:5000/nginx:latest
     ports:
     - containerPort: 80

---

# -------nginx-service--------- #
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
spec:
 type: NodePort
 ports:
 - port: 80
   targetPort: 80
   nodePort: 30001
 selector:
  app: nginx

```

```kubectl apply -f nginx.yaml```








####  记一下对kubernetes集群的搭建部署  

**ubantu下用kubeadm搭建kubernetes集群**  

[官方安装教程](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

1.  ubuntu + docker 环境  (目前是两个服务器组建集群server1+server2)

2. 安装kubelet kubeadm和kubectl  

- 安装 apt-transport-https  
``` # apt-get update && apt-get install -y apt-transport-https ```  

- 安装gpg证书(阿里镜像仓库的k8s)
``` # curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - ```  

- 更新软件源信息  
```` 
 # cat << EOF >/etc/apt/sources.list.d/kubernetes.list  
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main  
   EOF 
````
- 更新并安装kubelet kubeadm kubectl  
``` # apt-get update && apt-get install -y kubelet kubeadm kubectl ```  
``` 指定版本为: ```  
``` # apt-get update && apt-get install -y kubelet=11 kubeadm=11 kubectl=11 ```    

- 关闭swap  
``` sudo swapoff -a ```

3. master server1上初始化部署kubernetes的master  

- 获取初始化所需版本docker镜像，k8s=v1.11.1在我的docker hub 's tomsun28可以拉取  

```
k8s=v1.11.1所对应镜像及版本:
k8s.gcr.io/coredns:1.1.3
k8s.gcr.io/etcd-amd64:3.2.18
k8s.gcr.io/kube-apiserver-amd64:v1.11.1
k8s.gcr.io/kube-controller-manager-amd64:v1.11.1
k8s.gcr.io/kube-proxy-amd64:v1.11.1
k8s.gcr.io/kube-scheduler-amd64:v1.11.1
k8s.gcr.io/pause:3.1
```

- 初始化master  
``` kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16 ```  

- 成功之后会有join集群的脚步提示，记一下  
``` kubeadm join 192.168.0.3:6443 --token q6gmgt.3dakenwttapw4n2o --discovery-token-ca-cert-hash sha256:dbf69119e962456c239c5f7821ee9a0db46fb643fc40da8776d4e032de072085 ```  

- 根据output提示，to start using your cluster, you need to run(no root user )  
````
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
````
或者(root user)：  
``` export KUBECONFIG=/etc/kubernetes/admin.conf ```  


4. 安装 pod network 提供 pods 节点之前相互通信  

- 运行下面命令设置 ````/proc/sys/net/bridge/bridge-nf-call-iptables````为1  
```
sysctl net.bridge.bridge-nf-call-iptables=1
```

- 选择 flannel 作为 pod network  
``` 
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/c5d10c8/Documentation/kube-flannel.yml
```

- 要使 flannel 能正常使用,需要在master初始化时 kubeadm init 添加对应pod-network-cidr  
``` kubeadm init --pod-network-cidr=10.244.0.0/16 ```  

- 解除master不能调度运行其他pod的限制  
``` kubectl taint nodes --all node-role.kubernetes.io/master- ```  


5. server2上部署kebernetes并作为节点join to master  

- 在server2服务器上执行步骤2  

- 作为node节点加入到master集群中  
``` kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash> ```  

6. 在master上查看集群node节点分布  

``` kubectl get nodes ```

7. 对kubeadm所做的搭建进行undo  revert   

``` kubeadm reset ```  





**分享一波阿里云代金券[快速上云](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=rjlzz3uf&utm_source=rjlzz3uf)**
<br>

*参考来自*
[kubernetes官方部署文档](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)  
<br>
<br>
*转载请注明* [from tomsun28](http://usthe.com)  