---
layout: post
title:  kubernetes hpa
date: 2020-08-18
tag: 云原生
---

###  kubernetes hpa 实践

最近对公司一个项目进行了hpa能力集成，之前pod的扩容都是手工去修改，手工的维护当然没有自动化好，手工确定pod的数量很难平衡环境资源和系统性能之间的点，
遇到性能下降，固然拉几个pod马上解决问题，但之后的资源浪费也是存在的，当然为了省资源，把pod数量降到一个低水平，性能又提不上去。
hpa记得是k8s 1.6就出来了，最初了话只支持指标比较少cpu 内存,后面的版本慢慢支持了自定义指标单元。这里也就是我们下面要实践的hpa 自定义指标。
为什么使用监控自定义指标而使用原生支持的cpu 内存不能满足,对于业务pod来说，pod的横向自动伸缩锁所关注的指标除了pod本身的cpu内存外，
qps,队列大小等系列业务指标更能体现当前pod在一个什么样的负载水平，使用这些指标也能更好的贴合业务，流量处理数大时扩，不忙就缩。  

### 使用自定义指标进行 kubernetes hpa实践  

话不多说开始弄  

1. 部署指标采集adapter k8s, hpa是通过adapter来搜集数据的   

主流的或者说被很多文章写的k8s hpa 自定义指标 adapter 是 k8s-prometheus-adapter， 但这个并不是很适合我们，我们想要的是一个轻量的指标采集器adapter,
它可以去采集pod暴露出来的restful接口指标，也可以去采集外部接口指标，关键它很轻量。
刚好有个adapter很合适 - (zalando-incubator/kube-metrics-adapter](https://github.com/zalando-incubator/kube-metrics-adapter)  

开始部署  

kubectl apply -f adapter-deployment.yaml    
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-metrics-adapter
  namespace: kube-system
  labels:
    application: kube-metrics-adapter
    version: latest
spec:
  replicas: 1
  selector:
    matchLabels:
      application: kube-metrics-adapter
  template:
    metadata:
      labels:
        application: kube-metrics-adapter
        version: latest
    spec:
      serviceAccountName: custom-metrics-apiserver
      containers:
      - name: kube-metrics-adapter
        image: banzaicloud/kube-metrics-adapter:latest
        args:
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
````

kubectl apply -f adapter-rbac.yaml    
````

kind: ServiceAccount
apiVersion: v1
metadata:
  name: custom-metrics-apiserver
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-metrics-server-resources
rules:
- apiGroups:
  - custom.metrics.k8s.io
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: external-metrics-server-resources
rules:
- apiGroups:
  - external.metrics.k8s.io
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-metrics-resource-reader
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  - services
  verbs:
  - get
  - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-metrics-resource-collector
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list
- apiGroups:
  - apps
  resources:
  - deployments
  - statefulsets
  verbs:
  - get
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs:
  - get
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hpa-controller-custom-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: custom-metrics-server-resources
subjects:
- kind: ServiceAccount
  name: horizontal-pod-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hpa-controller-external-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-metrics-server-resources
subjects:
- kind: ServiceAccount
  name: horizontal-pod-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: custom-metrics-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: custom-metrics-apiserver
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: custom-metrics:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: custom-metrics-apiserver
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: custom-metrics-resource-collector
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: custom-metrics-resource-collector
subjects:
- kind: ServiceAccount
  name: custom-metrics-apiserver
  namespace: kube-system
````

kubectl apply -f custom-metrics-apiservice.yaml    
````

apiVersion: apiregistration.k8s.io/v1beta1
kind: APIService
metadata:
  name: v1beta1.custom.metrics.k8s.io
spec:
  service:
    name: kube-metrics-adapter
    namespace: kube-system
  group: custom.metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
````

kubectl apply -f adapter-service.yaml    
````

apiVersion: v1
kind: Service
metadata:
  name: kube-metrics-adapter
  namespace: kube-system
spec:
  ports:
  - port: 443
    targetPort: 443
  selector:
    application: kube-metrics-adapter
````


2. 部署需要进行自动伸缩的应用-其需要提供我们监控的rest接口   

kubectl apply -f deployment.yaml  

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-metrics-consumer
  labels:
    application: custom-metrics-consumer
    version: latest
spec:
  selector:
    matchLabels:
      application: custom-metrics-consumer
  template:
    metadata:
      labels:
        application: custom-metrics-consumer
        version: latest
    spec:
      containers:
      - name: custom-metrics-consumer
        image: mikkeloscar/custom-metrics-consumer:latest
        args:
        - --fake-queue-length=2000
        resources:
          limits:
            cpu: 10m
            memory: 25Mi
          requests:
            cpu: 10m
            memory: 25Mi
````

3. 部署管理伸缩上面那个应用的hpa  

kubectl apply -f hpa.yaml   
````
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-metrics-consumer
  namespace: custom-your-deployment-namesapce
  labels:
    application: custom-metrics-consumer
  annotations:
    # metric-config.<metricType>.<metricName>.<collectorName>/<configKey>
    metric-config.pods.queue-length.json-path/json-key: "$.queue.length"
    metric-config.pods.queue-length.json-path/path: /metrics
    metric-config.pods.queue-length.json-path/port: "9090"
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: custom-metrics-consumer
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: queue-length
      target:
        averageValue: 1k
        type: AverageValue
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

````

4. 弄完之后就可以开始对你的应用测试啦   


**分享一波阿里云代金券[快速上云](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=rjlzz3uf&utm_source=rjlzz3uf)**
<br>

<br>
<br>  

*转载请注明*  [from tomsun28](http://usthe.com)  
