# Kubernetes

Infrastructure as a service ->iaas

platform as a service -> paas

software as a service -> saas



mesos-> 分布式资源管理

docker swarm ->docker的集群资源管理

kubernetes：

* 轻量级：资源消耗小
* 开源
* 弹性伸缩
* 负载均衡：ipvs

![image-20210514144750511](imgs/Untitled.md)

资源清单:yaml

```yaml
appBersion: apps/v1 #api版本
kind: Deployment #资源类型
metadata: #资源元数据
 name: nginx-deployment
 namespace: default
spec: #资源规格
 replicas: 3 #副本数量
 selector: #标签选择器
  matchLabels:
   app: nginx
 template: #pod模板
  metadata: #pod元数据
   labels:
    app: nginx
  spec: #pod规格
   containers: #容器配置
   -name: nginx
    image: nginx:1.15
    ports:
```

组成部分

1. 控制器定义
2. 被控制的部分

https://www.huweihuang.com/kubernetes-notes/ 

