#### Kubernetes从入门到熟练应用

<p align="center">
<img width="260" align="center" src="../../images/11.jpg" />
</p>

当今三大主流调度系统的比较与分析,为何选择kubernetes作为调度系统:

目前的容器编排调度工具中kubernetes是领导者，在我们深入学习和探讨kubernetes之前，我们比较下当今三大主流调度系统:Docker Swarm, Kuberentes和Messos的不同之处看看为何kubernetes可以成为当今容器编排调度工具的领导者。

* Docker Swarm

Docker Swarm是Docker公司的容器编排系统,使用的是标准Docker API，容器使用命令和docker命令是一套，简单方便。Docker Swarm基本架构是也是简单直接，每个主机运行一个Docker Swarm代理，一个主机运行一个Docker Swarm管理者，这个管理者负责指挥和调度这些主机上的容器，Docker Swarm以高可用性模式运行，Docker Swarm中的一个节点充当其他节点的管理器，包括调度程序和服务发现组件的容器。 Docker Swarm的优点和缺点都是使用标准的Docker接口，因为使用简单，容易集成到现有系统，所以在支持复杂的调度系统时候就会比较困难了，特别是在定制的接口中实现的调度。这也许就是成也在Docker，败也在Docker的原因所在。

* Kubernetes

Kubernetes作为一个容器集群管理系统，用于管理云平台中多个主机上的容器的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一整套完整的机制。 Kubernetes没有固定要求容器的格式，但是Kubernetes使用它自己的API和命令行接口（CLI）来进行容器编排。 除了Docker容器之外, Kubernetes还支持其他多种容器，如rkt——最初由CoreOS创建，现在是Cloud Native Computing Foundation(CNCF)基金会托管的项目。 Kubernetes 是自成体系的管理工具，可以实现容器调度，资源管理，服务发现，健康检查，自动伸缩，更新升级等，也可以在应用模版配置中指定副本数量，服务要求（IO优先，性能优先等），资源使用区间，标签（Labels等）来匹配特定要求达到预期状态等，这些特征便足以征服开发者，再加上Kubernetes有一个非常活跃的社区。它为用户提供了更多的选择以方便用户扩展编排容器来满足他们的需求。但是由于Kubernetes使用了自己的API，所以命令系统是另外一套系统，这也是kubernetes门槛比较高的原因所在。

* Apache Mesos

Apache Mesos是一个分布式系统内核的开源集群管理器，Mesos的出现早于Docker Swarm和Kubernetes。再加上Marathon，一个用于基于容器的应用程序的编排框架，它为Docker Swarm和Kubernetes提供了一个有效的替代方案。Mesos同时可以使用其他框架来同时支持容器化和非容器化的工作负载。

Apache Mesos能够在同样的集群机器上运行多种分布式系统类型，可以更加动态高效的共享资源。而且Messos也提供服务失败检查，服务发布，服务跟踪，服务监控，资源管理和资源共享。Messos可以扩展伸缩到数千个节点。 如果你拥有很多的服务器而且想构建一个大的集群的时候，Mesos就派上用场了。很多的现代化可扩展性的数据处理应用都可以在Mesos上运行，包括大数据框架Hadoop、Kafka、Spark。 但是大而全，往往就是对应的复杂和困难，这一点体现在Messos上是完全正确,与Docker和Docker Swarm 使用同一种API不同的，Mesos和Marathon都有自己的API，这使得它们比其他编排系统更加的复杂。 Apache Mesos是混合环境的完美编配工具，由于它包含容器和非容器的应用,虽然Messos很稳定，但是它的使用户快速学习应用变得更加困难，这也是在应用和部署场景下难于推广的原因之一。

大部分的应用程序我们在部署的时候都会适当的添加监控，对于运行载体容器则更应该如此。kubernetes提供了 liveness probes来检查我们的应用程序。它是由节点上的kubelet定期执行的。
