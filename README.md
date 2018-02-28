# kubernetes-tutorial 

#### kubernetes 
Kubernetes是为生产环境而设计的容器调度管理系统，对于负载均衡、服务发现、高可用、滚动升级、自动伸缩等容器云平台的功能要求有原生支持。
<p align="center">
<img width="300" align="center" src="src/images/1.jpg" />
</p>

一个K8s集群是由分布式存储（etcd）、服务节点（Minion，etcd现在称为Node）和控制节点（Master）构成的。所有的集群状态都保存在etcd中，Master节点上则运行集群的管理控制模块。Node节点是真正运行应用容器的主机节点，在每个Minion节点上都会运行一个Kubelet代理，控制该节点上的容器、镜像和存储卷等。
