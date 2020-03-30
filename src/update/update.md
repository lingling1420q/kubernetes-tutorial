#### kubernetes集群手动更新版本

该文主要介绍如何用kubeadm 创建的 Kubernetes 集群从 1.15.2 版本升级到 1.16.1 版本.

通常我们使用 kubeadm 搭建的集群来更新是非常方便的，但是由于我们这里版本跨度太大，最新的版本是1.18.0,但是我们不能直接从 1.15.2更新到 1.18.0，kubeadm 的更新是不支持跨多个主版本的，所以我们现在是1.15.2，只能更新到 1.16.1 版本了，然后再重1.16.1 更新到 1.17.1 …… 不过版本更新的方式方法基本上都是一样的，所以后面要更新的话也挺简单了，下面我们就先将集群更新到  1.15.2版本。

#### 更新集群

首先我们需要保留 kubeadm config 文件：
```bash
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
```bash
> yum makecache fast && yum install -y kubeadm-1.15.2-0 kubectl-1.15.2-0 
> kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa71213468", GitTreeState:"clean", BuildDate:"2019-08-05T09:20:51Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

因为 kubeadm upgrade plan 命令执行过程中会去 dl.k8s.io 获取版本信息，这个地址是需要科学方法才能访问的，所以我们可以先将 kubeadm 更新到目标版本，然后就可以查看到目标版本升级的一些信息了。

执行 upgrade plan 命令查看是否可以升级：

```bash
> kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
I0518 18:50:12.844665    9676 feature_gate.go:230] feature gates: &{map[]}
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.15.2
[upgrade/versions] kubeadm version: v1.15.2
[upgrade/versions] WARNING: Couldn't fetch latest stable version from the internet: unable to get URL "https://dl.k8s.io/release/stable.txt": Get https://dl.k8s.io/release/stable.txt: dial tcp 35.201.71.162:443: i/o timeout
[upgrade/versions] WARNING: Falling back to current kubeadm version as latest stable version
[upgrade/versions] WARNING: Couldn't fetch latest version in the v1.10 series from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.10.txt": Get https://dl.k8s.io/release/stable-1.10.txt: dial tcp 35.201.71.162:443: i/o timeout

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.15.2   v1.15.2

Upgrade to the latest stable version:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.15.2   v1.15.2
Controller Manager   v1.15.2   v1.15.2
Scheduler            v1.15.2   v1.15.2
Kube Proxy           v1.15.2   v1.15.2
CoreDNS                        1.1.3
Kube DNS             v1.15.2
Etcd                 3.1.12    3.2.18

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.15.2
```

我们可以先使用 dry-run 命令查看升级信息：
```bash
> $ kubeadm upgrade apply v1.15.2 --config kubeadm-config.yaml --dry-run
```
注意要通过--config指定上面保存的配置文件，该配置文件信息包含了上一个版本的集群信息以及修改搞得镜像地址。

查看了上面的升级信息确认无误后就可以执行升级操作了：
```bash
> kubeadm upgrade apply v1.16.0 --config kubeadm-config.yaml
kubeadm upgrade apply v1.16.0  --config kubeadm-config.yaml
[preflight] Running pre-flight checks.
I0518 18:57:29.134722   12284 feature_gate.go:230] feature gates: &{map[]}
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration options from a file: kubeadm-config.yaml
I0518 18:57:29.179231   12284 feature_gate.go:230] feature gates: &{map[]}
[upgrade/apply] Respecting the --cri-socket flag that is set with higher priority than the config file.
[upgrade/version] You have chosen to change the cluster version to "v1.16.0"
[upgrade/versions] Cluster version: v1.15.2
[upgrade/versions] kubeadm version: v1.15.2 
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.16.0"...
Static pod: kube-apiserver-ydzs-master hash: 3abd7df4382a9b60f60819f84de40e11
Static pod: kube-controller-manager-ydzs-master hash: 1a0f3ccde96238d31012390b61109573
Static pod: kube-scheduler-ydzs-master hash: 2acb197d598c4730e3f5b159b241a81b
```

等几分钟时间看下,如果看到如下信息就证明集群升级成功了：
```bash
......
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS


[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.16.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```
由于上面我们已经更新过 kubectl 了，现在我们用 kubectl 来查看下版本信息：
```bash
> kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.16.0", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.16.0", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```




