# kubernetes-tutorial 

#### 搭建本地kubernetes集群
Kubernetes是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。

Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用（就像canary deployments）。

Kubernetes是为生产环境而设计的容器调度管理系统，对于负载均衡、服务发现、高可用、滚动升级、自动伸缩等容器云平台的功能要求有原生支持。

<p align="center">
<img width="700" align="center" src="src/images/1.jpg" />
</p>

一个K8s集群是由分布式存储（etcd）、服务节点（Minion，etcd现在称为Node）和控制节点（Master）构成的。所有的集群状态都保存在etcd中，Master节点上则运行集群的管理控制模块。Node节点是真正运行应用容器的主机节点，在每个Minion节点上都会运行一个Kubelet代理，控制该节点上的容器、镜像和存储卷等。

首先我们先来了解下Kubernetes中的主要概念：
1. Cluster : 集群是指由Kubernetes使用一系列的物理机、虚拟机和其他基础资源来运行你的应用程序。
2. Node : 一个node就是一个运行着Kubernetes的物理机或虚拟机，并且pod可以在其上面被调度.
3. Pod : 一个pod对应一个由相关容器和卷组成的容器组.
4. Label : 一个label是一个被附加到资源上的键/值对，譬如附加到一个Pod上，为它传递一个用户自定的并且可识别的属性.Label还可以被应用来组织和选择子网中的资源.
5. selector是一个通过匹配labels来定义资源之间关系得表达式，例如为一个负载均衡的service指定所目标Pod.
6. Replication Controller : replication controller 是为了保证一定数量被指定的Pod的复制品在任何时间都能正常工作.它不仅允许复制的系统易于扩展，还会处理当pod在机器在重启或发生故障的时候再次创建一个.
7. Service : 一个service定义了访问pod的方式，就像单个固定的IP地址和与其相对应的DNS名之间的关系。
8. Volume: 一个volume是一个目录，可能会被容器作为未见系统的一部分来访问。Kubernetes volume 构建在Docker Volumes之上,并且支持添加和配置volume目录或者其他存储设备.
9. Secret : Secret 存储了敏感数据，例如能允许容器接收请求的权限令牌。
10. Name : 用户为Kubernetes中资源定义的名字.
11. Namespace : Namespace 好比一个资源名字的前缀。它帮助不同的项目、团队或是客户可以共享cluster,例如防止相互独立的团队间出现命名冲突.
12. Annotation : 相对于label来说可以容纳更大的键值对，它对我们来说可能是不可读的数据，只是为了存储不可识别的辅助数据，尤其是一些被工具或系统扩展用来操作的数据.


#### 安装Docker 
<p align="center">
<img width="300" align="center" src="src/images/2.jpg" />
</p>
这里需要说明下Docker在2016年很早的时候就明确了将会在企业级方面重点跟进。而在短短的一年时间之内推出的1.12和1.13的版本在功能上确实是很大的进步。而在2017年的3月1号之后，Docker的版本命名开始发生变化，同时将CE版本和EE版本进行分开，而这些也是突然发现docker1.13的安装脚本不好用了才发现的，一起简单来看一下具体情况吧。

但是Docker企业版(EE)和Docker社区版(CE)版本有何不同呢？

1. Docker CE是简单的经典OSS Docker引擎。
  Docker Engine已经重新命名为Docker Community Edition，顾名思义，这是一个自己动手的，社区支持的Docker版本，免费提供。
  社区版将提供两个版本：Edge和Stable。 Edge将会每月发布最新的function。 稳定将按季度发布。 虽然Edge将会收到针对当前版本的安全更新和错误修复，但稳定版本将在初始版本发布后的四个月内得到类似的更新。 此更新周期将为用户提供足够大的窗口来计划从旧版本升级。

  虽然这两个版本都针对不同的受众，但在源代码级别上没有太大的差别。 墨西拿说：“Docker EE和CE都是基于开放源码的Docker项目，Docker项目是由Docker的合作伙伴和贡献者共同开发的，这就形成了所有Docker CE和EE版本的开放模块化核心。

2. Docker还提供了一个authentication计划来帮助第三方供应商确保他们的产品与Docker EE一起工作。
  Docker企业版有三个版本：基本版，标准版和高级版。 基本版附带Docker平台，支持和authentication，而标准版和高级版则增加了附加function，如容器pipe理（Docker Datacenter）和Docker安全扫描。

  Docker EE由阿里巴巴，Canonical，HPE，IBM，Microsoft和区域合作伙伴networking提供支持。 那些想testingDocker EE的人可以从官方网站免费下载试用版。

  Docker还提供了一个authentication计划来帮助第三方供应商确保他们的产品与Docker EE一起工作。


Docker EE是Docker CE，在某些系统上获得authentication，并由Docker Inc.提供支持。
<p align="center">
<img width="600" align="center" src="src/images/3.jpg" />
</p>
实际上Docker从17.03开始分为企业版与社区版，社区版并非阉割版，而是改了个名称；企业版则提供了一些收费的高级特性。EE版本维护期1年；CE的stable版本三个月发布一次，维护期四个月；另外CE还有edge版，一个月发布一次。个人用社区版开发完全可以满足开发要求！

首先安装docker环境，这个可以根据电脑系统的不同，选择不同的安装方式。
[mac安装]()
[docker安装文档](https://docs.docker.com/install/)
