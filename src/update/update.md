#### kubernetes集群手动更新版本

该文主要介绍如何用kubeadm 创建的 Kubernetes 集群从 1.15.2 版本升级到 1.16.1 版本.

通常我们使用 kubeadm 搭建的集群来更新是非常方便的，但是由于我们这里版本跨度太大，最新的版本是1.18.0,但是我们不能直接从 1.15.2更新到 1.18.0，kubeadm 的更新是不支持跨多个主版本的，所以我们现在是1.15.2，只能更新到 1.16.1 版本了，然后再重1.16.1 更新到 1.17.1 …… 不过版本更新的方式方法基本上都是一样的，所以后面要更新的话也挺简单了，下面我们就先将集群更新到  1.15.2版本。

#### 更新集群

首先我们需要保留 kubeadm config 文件：
```yaml
> kubeadm config view
api:
  advertiseAddress: 10.151.30.11
  bindPort: 6443
  controlPlaneEndpoint: ""
auditPolicy:
  logDir: /var/log/kubernetes/audit
  logMaxAge: 2
  path: ""
authorizationModes:
- Node
- RBAC
certificatesDir: /etc/kubernetes/pki
cloudProvider: ""
criSocket: /var/run/dockershim.sock
etcd:
  caFile: ""
  certFile: ""
  dataDir: /var/lib/etcd
  endpoints: null
  image: ""
  keyFile: ""
imageRepository: k8s.gcr.io
kubeProxy:
  config:
    bindAddress: 0.0.0.0
    clientConnection:
      acceptContentTypes: ""
      burst: 10
      contentType: application/vnd.kubernetes.protobuf
      kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
      qps: 5
    clusterCIDR: 10.244.0.0/16
    configSyncPeriod: 15m0s
    conntrack:
      max: null
      maxPerCore: 32768
      min: 131072
      tcpCloseWaitTimeout: 1h0m0s
      tcpEstablishedTimeout: 24h0m0s
    enableProfiling: false
    healthzBindAddress: 0.0.0.0:10256
    hostnameOverride: ""
    iptables:
      masqueradeAll: false
      masqueradeBit: 14
      minSyncPeriod: 0s
      syncPeriod: 30s
    ipvs:
      minSyncPeriod: 0s
      scheduler: ""
      syncPeriod: 30s
    metricsBindAddress: 127.0.0.1:10249
    mode: ""
    nodePortAddresses: null
    oomScoreAdj: -999
    portRange: ""
    resourceContainer: /kube-proxy
    udpIdleTimeout: 250ms
kubeletConfiguration: {}
kubernetesVersion: v1.10.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
nodeName: keke-master
privilegedPods: false
token: ""
tokenGroups:
- system:bootstrappers:kubeadm:default-node-token
tokenTTL: 24h0m0s
tokenUsages:
- signing
- authentication
unifiedControlPlaneImage: ""
```

将上面的imageRepository值更改为：`gcr.azk8s.cn/google_containers`，然后保存内容到文件 kubeadm-config.yaml 中（当然如果你的集群可以获取到 grc.io 的镜像可以不用更改）.


然后更新 kubeadm:
```yaml
> yum makecache fast && yum install -y kubeadm-1.15.2-0 kubectl-1.15.2-0 
> kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa71213468", GitTreeState:"clean", BuildDate:"2019-08-05T09:20:51Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

```markdown
因为 kubeadm upgrade plan 命令执行过程中会去 dl.k8s.io 获取版本信息，这个地址是需要科学方法才能访问的，所以我们可以先将 kubeadm 更新到目标版本，然后就可以查看到目标版本升级的一些信息了。
```

