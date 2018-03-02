# kubernetes pods

在Kubernetes中，能够被创建、调度和管理的最小部署单元是Pod，而非单个容器。

<p align="center">
<img width="700" align="center" src="../images/8.jpg" />
</p>

Pod里的容器是共享网络和存储的。

Pods包含多个容器，是连接在一起的容器组合并共享文件卷。它们是最小的部署单元，由 Kubernetes 统一创建、调度、管理。Pods是可以直接创建的，但推荐的做法是使用 Replication Controller，即使是创建一个 Pod。

一个Pod可以被一个容器化的环境看做是应用层的逻辑宿主机(Logical Host)，通常一个Node中可以运行几百个Pod，每个Pod中有多个容器应用，同一个Pod中的多个容器应用通常是紧密耦合的(相当于多个业务容器组成的一个逻辑虚拟机)。

一个Pod中的多个容器应用通常是紧耦合的。Pod在Node上被创建、启动或者销毁。

每个Pod中有一个特殊的Pause容器，其他的成为业务容器，这些业务容器共享Pause容器的网络栈以及Volume挂载卷，因而他们之间的通信及数据交互更为高效。

同一个pod中的业务容器共享如下资源：

* PID命名空间(不同应用程序可以看到其他应用程序的PID)
* 网络命名空间(pod中多个容器可以访问同一个IP和端口范围)
* IPC命名空间(能够使用SystemV IPC或者POSIX消息队列进行通信)
* UTS命名空间(共享同一个主机名)
* Volumes(访问定义在pod级别的存储卷)

Pod可以单独创建。由于Pods没有可控的生命周期，如果他们进程死掉了，他们将不会重新创建。出于这个原因，建议您使用复制控制器。









































License
This is free software distributed under the terms of the MIT license
