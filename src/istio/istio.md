#### kubernetes安装运行Istio应用

2017年5月，Google、IBM和Lyft发布了开源服务网格框架Istio，提供微服务的连接、管理、监控和安全保护。Istio提供了一个服务间通信的基础设施层，解耦了应用逻辑和服务访问中版本管理、安全防护、故障转移、监控遥测等切面的问题。

Istio为希腊语，意思是“启航”，虽然是一个非常年轻的项目却得到了极大的关注，其生态发展非常迅猛。

Istio 是一个开放式平台，可用于连接、管理和保护微服务。 它为您提供了一种简单的方法来创建已部署服务（包括负载均衡、服务到服务认证、监视等等）的网络，而无需对服务代码进行任何更改。 要对服务添加 Istio 支持，您必须在整个环境中部署一个特殊的侧柜代理，该代理通过使用 Istio 中提供的控制平面功能来拦截微服务（已配置的微服务和受管微服务）之间的所有网络通信。

<p align="center">
<img width="600" align="center" src="../images/66.jpg" />
</p>

#### 安装 Istio

在安装Istio之前希望打击可以按照前面kubernetes集群文章搭建好kubernetes集群，然后开始使用Istio.

这里需要下载最新版的istio:
```bash
> curl -L https://git.io/getLatestIstio | sh -
```
然后把istio环境变量添加到系统中:

```bash
> cd istio-1.0.0/
> export PATH=$PWD/bin:$PATH
```
其中安装文件在install目录下，istioctl执行文件在bin目录下，一些应用文件在samples目录下。

我们可以先安装下istio的应用:
```bash
> kubectl apply -f install/kubernetes/istio-demo.yaml
```
运行查看service和pod:
```bash
> kubectl get pod -n istio-system
NAME                                        READY     STATUS    RESTARTS   AGE
grafana-6dd4cb7ffd-n87q4                    1/1       Running   0          2d
istio-citadel-b874fd9f5-kk6vs               1/1       Running   0          2d
istio-egressgateway-ddcdd644c-6ppq4         1/1       Running   0          2d
istio-egressgateway-ddcdd644c-7kgrc         1/1       Running   0          2d
istio-egressgateway-ddcdd644c-9n2df         1/1       Running   0          2d
istio-egressgateway-ddcdd644c-bx94h         1/1       Running   0          2d
istio-egressgateway-ddcdd644c-sd2pj         1/1       Running   0          2d
istio-galley-8985546b8-lblnm                1/1       Running   0          2d
istio-ingressgateway-7565c689cb-52zdw       1/1       Running   0          2d
istio-ingressgateway-7565c689cb-czvgb       1/1       Running   0          2d
istio-ingressgateway-7565c689cb-gm4w8       1/1       Running   0          2d
istio-ingressgateway-7565c689cb-pqxlb       1/1       Running   0          2d
istio-ingressgateway-7565c689cb-vqxhg       1/1       Running   0          2d
istio-pilot-58b5d5f-mvzrr                   2/2       Running   0          2d
istio-policy-686ff55f4f-kl4hn               2/2       Running   0          2d
istio-policy-686ff55f4f-l5q8d               2/2       Running   0          2d
istio-sidecar-injector-5d4b7b4957-lpfkr     1/1       Running   0          2d
istio-statsd-prom-bridge-58f8596c67-tfbwx   1/1       Running   0          2d
istio-telemetry-6bff9755fd-pkht9            2/2       Running   0          2d
istio-tracing-75d76fb9f-mzjpf               1/1       Running   0          2d
prometheus-884dbbcd5-p7wv7                  1/1       Running   0          2d
servicegraph-646bbc8cb4-6kvdb               1/1       Running   0          2d
```
```bash
> > kubectl get svc -n istio-system
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                                                                                     AGE
grafana                    ClusterIP      10.43.125.188   <none>          3000/TCP                                                                                                    2d
istio-citadel              ClusterIP      10.43.84.200    <none>          8060/TCP,9093/TCP                                                                                           2d
istio-egressgateway        ClusterIP      10.43.97.201    <none>          80/TCP,443/TCP                                                                                              2d
istio-galley               ClusterIP      10.43.127.149   <none>          443/TCP,9093/TCP                                                                                            2d
istio-ingressgateway       LoadBalancer   10.43.120.83    120.92.172.35   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:30182/TCP,8060:30819/TCP,15030:32142/TCP,15031:31067/TCP   2d
istio-pilot                ClusterIP      10.43.216.64    <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                       2d
istio-policy               ClusterIP      10.43.45.54     <none>          9091/TCP,15004/TCP,9093/TCP                                                                                 2d
istio-sidecar-injector     ClusterIP      10.43.37.12     <none>          443/TCP                                                                                                     2d
istio-statsd-prom-bridge   ClusterIP      10.43.93.131    <none>          9102/TCP,9125/UDP                                                                                           2d
istio-telemetry            ClusterIP      10.43.69.34     <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                       2d
jaeger-agent               ClusterIP      None            <none>          5775/UDP,6831/UDP,6832/UDP                                                                                  2d
jaeger-collector           ClusterIP      10.43.90.61     <none>          14267/TCP,14268/TCP                                                                                         2d
jaeger-query               ClusterIP      10.43.150.244   <none>          16686/TCP                                                                                                   2d
prometheus                 ClusterIP      10.43.142.240   <none>          9090/TCP                                                                                                    2d
servicegraph               ClusterIP      10.43.224.250   <none>          8088/TCP                                                                                                    2d
tracing                    ClusterIP      10.43.7.11      <none>          80/TCP                                                                                                      2d
zipkin                     ClusterIP      10.43.136.215   <none>          9411/TCP                                                                                                    2d
```

