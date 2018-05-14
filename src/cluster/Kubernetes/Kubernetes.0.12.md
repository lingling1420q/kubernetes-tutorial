#### Kubernetes 

Kubernetes官方文档:[https://kubernetes.io/docs/reference/](https://kubernetes.io/docs/reference/)
Kubernetes官方Git地址:[https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)

本系列中使用 KubernetesV1.8 RancherV1.6.14  


#### Kubectl命令

#### 对整个namespce进行资源限制

#### 权限验证

#### 弹性扩容实践


#### NodePort方式暴露服务的端口的默认范围（30000-32767）修改
则在apiserver的启动命令里面添加如下参数 –service-node-port-range=1-65535
