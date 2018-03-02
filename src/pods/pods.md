#### service 


Service是kubernetes最核心的概念，通过创建Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址，并且将请求进行负载分发到后端的各个容器应用上。

Service服务是一个虚拟概念，逻辑上代理后端pod。众所周知，pod生命周期短，状态不稳定，pod异常后新生成的pod ip会发生变化，之前pod的访问方式均不可达。通过service对pod做代理，service有固定的ip和port，ip:port组合自动关联后端pod，即使pod发生改变，kubernetes内部更新这组关联关系，使得service能够匹配到新的pod。这样，通过service提供的固定ip，用户再也不用关心需要访问哪个pod，以及pod是否发生改变，大大提高了服务质量。如果pod使用rc创建了多个副本，那么service就能代理多个相同的pod，通过kube-proxy，实现负载均衡。

集群中每个Node节点都有一个组件kube-proxy，实际上是为service服务的，通过kube-proxy，实现流量从service到pod的转发，kube-proxy也可以实现简单的负载均衡功能。

kube-proxy代理模式：userspace方式。kube-proxy在节点上为每一个服务创建一个临时端口，service的IP:port过来的流量转发到这个临时端口上，kube-proxy会用内部的负载均衡机制（轮询），选择一个后端pod，然后建立iptables，把流量导入这个pod里面。


#### Service定义详解
yaml格式的Service定义文件的完整内容：

```bash
apiVersion: v1
kind: Service
matadata:
  name: string
  namespace: string
  labels:
    - name: string
  annotations:
    - name: string
spec:
  selector: []
  type: string
  clusterIP: string
  sessionAffinity: string
  ports:
  - name: string
    protocol: string
    port: int
    targetPort: int
    nodePort: int
  status:
    loadBalancer:
      ingress:
        ip: string
        hostname: string
```
<p align="center">
<img width="700" align="center" src="../images/6.jpg" />
</p>


#### 负载分发策略
目前kubernetes提供了两种负载分发策略：RoundRobin和SessionAffinity.

RoundRobin：轮询模式，即轮询将请求转发到后端的各个Pod上.

SessionAffinity：基于客户端IP地址进行会话保持的模式，第一次客户端访问后端某个Pod，之后的请求都转发到这个Pod上.

默认是RoundRobin模式.

在某些场景中，开发人员希望自己控制负载均衡的策略，不使用Service提供的默认负载，kubernetes通过Headless Service的概念来实现。不给Service设置ClusterIP（无入口IP地址）：

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
  clusterIP: None
  selector:
    app: nginx
```

有时候，一个容器应用提供多个端口服务：

```bash
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  ports:
  - port: 8080
    targetPort: 8080
    name: web
  - port: 8005
    targetPort: 8005
    name: management
  selector:
    app: webapp
```
为不同的应用分配各自的端口。

另一个例子是两个端口使用了不同的4层协议，即TCP或UDP.

```bash
apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "KubeDNS"
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 169.169.0.100
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
```

#### 集群外部访问Pod或Service
为了让外部客户端可以访问这些服务，可以将Pod或者Service的端口号映射到宿主主机，使得客户端应用能够通过物理机访问容器应用。

将容器应用的端口号映射到物理机.通过设置容器级别的hostPort，将容器应用的端口号映射到物理机上：

pod-hostport.yaml：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  containers:
  - name: webapp
    image: tomcat
    ports:
    - containerPort: 8080
      hostPort:8081
```

通过kubectl create创建这个Pod：
```bash
kubectl create -f pod-hostport.yaml
```
通过物理机的IP地址和8081端口号访问Pod内的容器服务：

```bash
curl 10.0.11.151:8081
```
#### 通过设置Pod级别的hostNetwork=true
通过设置Pod级别的hostNetwork=true，该Pod中所有容器的端口号都将被直接映射到物理机上，设置hostWork=true是需要注意，在容器的ports定义部分如果不指定hostPort，则默认hostPort等于containerPort，如果指定了hostPort，则hostPort必须等于containerPort的值。

pod-hostnetwork.yaml：
```bash
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  hostNetwork: true
  containers:
  - name: webapp
    image: tomcat
    imagePullPolicy: Never
    ports:
    - containerPort: 8080
```
创建这个Pod：
```bash
kubectl create -f pod-hostnetwork.yaml
```

通过物理机的IP地址和8080端口访问Pod的容器服务：
```bash
curl 10.0.11.151:8080
```
#### 将Service的端口号映射到物理机

通过设置nodePort映射到物理机，同时设置Service的类型为NodePort：

webapp-svc-nodeport.yaml：
```bash
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 8081
  selector:
    app: webapp
```
创建这个Service：
```bash
kubectl create -f webapp-svc-nodeport.yaml
```

通过物理机的IP和端口访问：
```bash
curl 10.0.11.151:8081
```
如果访问不通，查看下物理机的防火墙设置。同样，对该Service的访问也将被负载分发到后端多个Pod上.



License
This is free software distributed under the terms of the MIT license
