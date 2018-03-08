#### arrange

在容器环境中，编排通常涉及到三个方面:

资源编排 - 负责资源的分配，如限制 namespace 的可用资源，scheduler 针对资源的不同调度策略； 

工作负载编排 - 负责在资源之间共享工作负载，如 Kubernetes 通过不同的 controller 将 Pod 调度到合适的 node 上，并且负责管理它们的生命周期；

服务编排 - 负责服务发现和高可用等，如 Kubernetes 中可用通过 Service 来对内暴露服务，通过 Ingress 来对外暴露服务。

在 Kubernetes 中有 5 种我们经常会用到的控制器来帮助我们进行容器编排，它们分别是 Deployment, StatefulSet, DaemonSet, CronJob, Job。















































License
This is free software distributed under the terms of the MIT license
