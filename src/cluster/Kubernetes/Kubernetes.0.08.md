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

大部分的应用程序我们在部署的时候都会适当的添加监控，对于运行载体容器则更应该如此。kubernetes提供了 liveness probes来检查我们的应用程序。它是由节点上的kubelet定期执行的。

## Pod的整个生命阶段：

#### Pending
表示集群系统正在创建Pod，但是Pod中的container还没有全部被创建，这其中也包含集群为container创建网络，或者下载镜像的时间；

#### Running
表示pod已经运行在一个节点商量，并且所有的container都已经被创建。但是并不代表所有的container都运行，它仅仅代表至少有一个container是处于运行的状态或者进程出于启动中或者重启中；

#### Succeeded
所有Pod中的container都已经终止成功，并且没有处于重启的container；
#### Failed
所有的Pod中的container都已经终止了，但是至少还有一个container没有被正常的终止(其终止时的退出码不为0)

### 对于liveness 
probes的结果也有几个固定的可选项值：


```
Success：表示通过检测

Failure：表示没有通过检测

Unknown：表示检测没有正常进行
```


## Liveness Probe的种类：

#### ExecAction
在container中执行指定的命令。当其执行成功时，将其退出码设置为0；

#### TCPSocketAction
执行一个TCP检查使用container的IP地址和指定的端口作为socket。如果端口处于打开状态视为成功；

#### HTTPGetAcction
执行一个HTTP默认请求使用container的IP地址和指定的端口以及请求的路径作为url，用户可以通过host参数设置请求的地址，通过scheme参数设置协议类型(HTTP、HTTPS)如果其响应代码在200~400之间，设为成功。
当前kubelet拥有两个检测器，他们分别对应不通的触发器(根据触发器的结构执行进一步的动作)：


## Liveness Probe
表示container是否处于live状态。如果 LivenessProbe失败，LivenessProbe将会通知kubelet对应的container不健康了。随后kubelet将kill掉 container，并根据RestarPolicy进行进一步的操作。默认情况下LivenessProbe在第一次检测之前初始化值为 Success，如果container没有提供LivenessProbe，则也认为是Success；

## ReadinessProbe：
表示container是否以及处于可接受service请求的状态了。如 果ReadinessProbe失败，endpoints controller将会从service所匹配到的endpoint列表中移除关于这个container的IP地址。因此对于Service匹配到的 endpoint的维护其核心是ReadinessProbe。默认Readiness的初始值是Failure，如果一个container没有提供 Readiness则被认为是Success。
对于LivenessProbe和ReadinessProbe用法都一样，拥有相同的参数和相同的监测方式。
initialDelaySeconds：用来表示初始化延迟的时间，也就是告诉监测从多久之后开始运行，单位是秒
timeoutSeconds: 用来表示监测的超时时间，如果超过这个时长后，则认为监测失败
当前对每一个Container都可以设置不同的restartpolicy，有三种值可以设置：
Always: 只要container退出就重新启动
OnFailure: 当container非正常退出后重新启动
Never: 从不进行重新启动
如果restartpolicy没有设置，那么默认值是Always。如果container需要重启，仅仅是通过kubelet在当前节点进行container级别的重启。
最后针对LivenessProbe如何使用，请看下面的几种方式，如果要使用ReadinessProbe只需要将livenessProbe修改为readinessProbe即可：

```
apiVersion: v1
kind: Pod
metadata:
  name: probe-exec
  namespace: coocla
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/health
      initialDelaySeconds: 5
      timeoutSeconds: 1
---
apiVersion: v1
kind: Pod
metadata:
  name: probe-http
  namespace: coocla
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
        host: www.baidu.com
        scheme: HTTPS
      initialDelaySeconds: 5
      timeoutSeconds: 1
---
apiVersion: v1
kind: Pod
metadata:
  name: probe-tcp
  namespace: coocla
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      initialDelaySeconds: 5
      timeoutSeconds: 1
      tcpSocket:
        port: 80
```


我们使用上面的construct创建资源：

```
kubectl create -f probe.yaml
kubectl get event
```

通过event我们可以发现对应的container由于probe监测失败一直处于循环重启中，其事件原因：unhealthy

