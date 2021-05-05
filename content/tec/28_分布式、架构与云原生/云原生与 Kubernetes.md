# 为什么要云原生


# 什么是云原生


## 云原生十二要素


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1593571295049-16176836-0e6b-40b8-9219-88266598be77.png#align=left&display=inline&height=295&margin=%5Bobject%20Object%5D&name=image.png&originHeight=589&originWidth=555&size=407793&status=done&style=none&width=277.5)
## 随笔记录

- 滚动更新过程中，terminating的pod回先被移出service，执行prestop后退出才会真的退出。如果prestop终止服务运行，那此时该pod是可以正确接收用户请求并处理的。（17.2.4）



# Container 容器


## 资源隔离实现

- Linux Namespace
- Cgroup



> Two mechanisms make this possible. The first one, Linux Namespaces, makes sure each process sees its own personal view of the system (files, processes, network interfaces, hostname, and so on). The second one is Linux Control Groups (cgroups), which limit the amount of resources the process can consume (CPU, memory, network bandwidth, and so on).



## 容器镜像的分层
> But layers don’t only make distribution more efficient, they also help reduce the storage footprint of images. Each layer is only stored once. Two containers created from two images based on the same base layers can therefore read the same files, but if one of them writes over those files, the other one doesn’t see those changes. Therefore, even if they share files, they’re still isolated from each other. **This works because container image layers are read-only. When a container is run, a new writable layer is created on top of the layers in the image. When the process in the container writes to a file located in one of the underlying layers, a copy of the whole file is created in the top-most layer and the process writes to the copy.**



## 目前缺陷
> In theory, a container image can be run on any Linux machine running Docker, but one small caveat exists—one related to the fact that all containers running on a host use the host’s Linux kernel. If a containerized application requires a specific kernel version, it may not work on every machine. If a machine runs a different version of the Linux kernel or doesn’t have the same kernel modules available, the app can’t run on it.



# Kubernetes 概览


## 架构
> At the hardware level, a Kubernetes cluster is composed of many nodes, which can be split into two types:
> - The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
> - Worker nodes that run the actual applications you deploy
> 


> Figure 1.9 shows the components running on these two sets of nodes.
> 

> ![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1597632396621-bdd78f4f-7302-4cb5-b3d2-26546f87a80b.png#align=left&display=inline&height=340&margin=%5Bobject%20Object%5D&name=image.png&originHeight=680&originWidth=1466&size=99085&status=done&style=none&width=733)



### MasterNode
> The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. These components are
> 

> - The Kubernetes API Server, which you and the other Control Plane components communicate with
> - The Scheduler, which schedules your apps (assigns a worker node to each deployable component of your application)
> - The Controller Manager, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on  etcd, a reliable distributed data store that persistently stores the cluster configuration.



### WorkerNode
> The worker nodes are the machines that run your containerized applications. The task of running, monitoring, and providing services to your applications is done by the following components:
> - Docker, rkt, or another container runtime, which runs your containers
> - The Kubelet, which talks to the API server and manages containers on its node
> - The Kubernetes Service Proxy (kube-proxy), which load-balances network traffic between application components



## 与 K8s 交互
![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1597632851502-5a63dbd4-88a3-40bf-9bc4-9a05b1e22bf9.png#align=left&display=inline&height=591&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1182&originWidth=1448&size=202766&status=done&style=none&width=724)




# Pod
> A pod is a group of one or more tightly related containers that will always run together on the same worker node and in the same Linux namespace(s).



# Controller


# Service / Ingress


# Volum


# ConfigMap / Secret


# Depolyment / StatefulSet




# Ref

- Kubernetes in Action (2017)
- Microservice Patterns (2017)
- Cloud Native Patterns (2019)
- [https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/definition](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/definition)
