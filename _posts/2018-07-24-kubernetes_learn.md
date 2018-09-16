---
layout: post
title:  kubernetes learn
date: 2018-07-24
tag: 分布式
---

###  kubernetes  

#### kubernetes 知识点  

1. 核心组件  

- etcd 保存整个集群的状态信息，感觉相当于k8s的数据库  
- apiserver 提供对k8s资源操作的唯一入口，并提供认证授权，访问控制，API注册与发现等机制  
- controller manager 负责维护集群的状态，eg:故障检测，自动扩展pod，滚动更新等  
- scheduler 负责对资源的调度，按着预定的调度策略将pod调度到相应的集群上  
- kubelet 负责维护容器的生命周期，相当于在node上的agent，负责管理pods和它们上面的容器，images镜像、volumes等  
- kube-proxy 负责为service提供集群内部的服务发现和负载均衡  




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
``` 指定版本为: # apt-get update && apt-get install -y kubelet=11 kubeadm=11 kubectl=11 ```  

3. master server1上初始化部署kubernetes的master  

- 获取初始化所需版本docker镜像，k8s=v1.11.1在我的docker hub 's tomsun28可以拉取  

```
k8s=v1.11.1所对应镜像及版本:
k8s.gcr.io_coredns:1.1.3
k8s.gcr.io_etcd-amd64:3.2.18
k8s.gcr.io_kube-apiserver-amd64:v1.11.1
k8s.gcr.io_kube-controller-manager-amd64:v1.11.1
k8s.gcr.io_kube-proxy-amd64:v1.11.1
k8s.gcr.io_kube-scheduler-amd64:v1.11.1
k8s.gcr.io_pause:3.1
```

- 初始化master  
``` kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16 ```  

- 成功之后会有join集群的脚步提示，记一下  
``` kubeadm join 192.168.0.3:6443 --token q6gmgt.3dakenwttapw4n2o --discovery-token-ca-cert-hash sha256:dbf69119e962456c239c5f7821ee9a0db46fb643fc40da8776d4e032de072085 ```  

- 根据output提示，可以配置非root用户也能run kubectl的命令行  
````
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
````
或者：  
``` export KUBECONFIG=/etc/kubernetes/admin.conf ```  

- 解除master不能调度运行其他pod的限制  
``` kubectl taint nodes --all node-role.kubernetes.io/master- ```  

4. 安装 pod network 提供 pods 节点之前相互通信  

- 选择 flannel 作为 pod network  
``` kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml ```  

- 要使 flannel 能正常使用,需要在master初始化时 kubeadm init 添加对应pod-network-cidr  
``` kubeadm init --pod-network-cidr=10.244.0.0/16 ```  

5. server2上部署kebernetes并作为节点join to master  

- 在server2服务器上执行步骤2  

- 作为node节点加入到master集群中  
``` kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash> ```  

6. 在master上查看集群node节点分布  

``` kubectl get nodes ```





**分享一波阿里云代金券[快速上云](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=rjlzz3uf&utm_source=rjlzz3uf)**
<br>

*参考来自*
[kubernetes官方部署文档](https://my.oschina.net/jayqqaa12/blog/633683?p=1&temp=1516212821799#blog-comments-list)  
<br>
<br>
*转载请注明* [from tomsun28](http://usthe.com)  
