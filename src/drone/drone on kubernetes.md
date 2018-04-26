#### Drone部署到k8s集群

本身drone这种基于容器的pipeline方式，和k8s是相当契合的。这样的好处有：
* k8s集群守护drone-server 和drone-agent。
* 可以利用rpc特性，根据agent负载压力来动态调整agent的数量。当然即使不动态调整，我们手动调整一下复制集的数目也是相当简单的。
* 部署到k8s集群以后，可以利用k8s已有的日志系统和监控系统。


#### 配置内容

ConfigMap在这里为drone应用的配置文件。这里有关于server和agent一系列设置。不过在k8s中大家需要注意的是：更新configmap以后，对于挂载该configmap的应用，配置内容并不能立即生效，大约需要10s。
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: drone-config
  namespace: devops
data:
  # Drone Server Config

  # server host name
  server.host: drone.xxx.com
  # start the server in debug mode
  server.debug: "false"
  # open user registration
  server.open: "true"
  # database driver, defaul as sqlite3
  server.database.driver: sqlite3
  # database driver configuration string
  server.database.datasource: drone.sqlite

  # remote parameters (Gogs)
  server.remote.gogs: "true"
  server.remote.gogs.url: "http://gogs.xxx.com"
  server.remote.gogs.private.mode: "true"

  # Drone Agent Config 
  agent.debug: "false"
  agent.debug.pretty: "false"
  agent.max.procs: "1"
  agent.healthcheck: "true"

```
