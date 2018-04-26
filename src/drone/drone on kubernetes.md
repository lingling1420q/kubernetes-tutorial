#### Drone部署到k8s集群

本身drone这种基于容器的pipeline方式，和k8s是相当契合的。这样的好处有：
* k8s集群守护drone-server 和drone-agent。
* 可以利用rpc特性，根据agent负载压力来动态调整agent的数量。当然即使不动态调整，我们手动调整一下复制集的数目也是相当简单的。
* 部署到k8s集群以后，可以利用k8s已有的日志系统和监控系统。

drone引入pipline的概念，整个build过程由多个stage组成，每一个stage都是docker。
* 各stage间可以通过共享宿主机的磁盘目录, 实现build阶段的数据共享和缓存基础数据, 实现加速下次build的目标
* 各stage也可以共享宿主机的docker环境，实现共享宿主机的docker image, 不用每次build都重新拉取base image，减少build时间
* 可以并发运行。多个build可以并发运行，单机并发数量由服务器cpu数决定。 由开发者负责打包image和流程控制。Docker-in-docker,这一点非常重要，一切都在掌握之中。相比jenkins的好处是，所有的image都是开发者提供，不需要运维参与在CI服务器上部署各种语言编译需要的环境。

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
Secret文件，主要是存放一些秘钥之类的。不过这里也是有坑的，这个secret用于server和angent通信，设置不对就会构建项目一直处于pending状态。切记k8s中，secret需要base64。

```bash
echo -n "yourpassword" | base64
eW91cnBhc3N3b3Jk
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: drone-secrets
  namespace: devops
data:
  server.secret: eW91cnBhc3N3b3Jk

```
drone-server的Deployment和Service和Ingress。此处为了简单，用了sqlite数据库，真正生产环境建议用mysql或是pgsql。
即使用sqlite，也应该挂载到ceph中，保证数据的安全。这里直接hostpath。kubernetes中，应该做到存储和计算的分离。

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: drone-server
  namespace: devops
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: drone-server
    spec:
      nodeSelector:
        net-type: external
      containers:
      - image: drone/drone:latest
        imagePullPolicy: Always
        name: drone-server
        ports:
        - containerPort: 8000
          protocol: TCP
        - containerPort: 9000
          protocol: TCP
        volumeMounts:
          # Persist our configs in an SQLite DB in here
          - name: drone-server-sqlite-db
            mountPath: /var/lib/drone
        resources:
          requests:
            cpu: 40m
            memory: 32Mi
        env:
        - name: DRONE_HOST
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.host
        - name: DRONE_OPEN
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.open
        - name: DRONE_DATABASE_DRIVER
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.database.driver
        - name: DRONE_DATABASE_DATASOURCE
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.database.datasource
        - name: DRONE_SECRET
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: server.secret
        - name: DRONE_GOGS
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.remote.gogs
        - name: DRONE_GOGS_URL
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.remote.gogs.url
        - name: DRONE_GOGS_PRIVATE_MODE
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.remote.gogs.private.mode
        - name: DRONE_DEBUG
          valueFrom:
            configMapKeyRef:
              name: drone-config
              key: server.debug
      volumes:
        - name: drone-server-sqlite-db
          hostPath:
            path: /var/lib/drone
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: drone-service
  namespace: devops
spec:
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8000
  - name: grpc
    protocol: TCP
    port: 9000
    targetPort: 9000
  selector:
    app: drone-server
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: drone-ingress
  namespace: devops
spec:
  rules:
  - host: drone.xxx.com
    http:
      paths:
      - backend:
          serviceName: drone-service
          servicePort: 80
        path: /
```

agent的部署文件了，replicas: 1 该项可以设置agent的数量，扩容起来特别方便。server和agent通过grpc的方式进行通信，主要端口是9000。
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: drone-agent
  namespace: devops
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: drone-agent
    spec:
      nodeSelector:
        net-type: external
      containers:
      - image: drone/agent:latest
        imagePullPolicy: Always
        name: drone-agent
        volumeMounts:
          # Enables Docker in Docker
          - name: docker-socket
            mountPath: /var/run/docker.sock
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
        livenessProbe:
          httpGet:
            path: /healthz
            port: 3000
          initialDelaySeconds: 3
          periodSeconds: 3
        env:
        - name: DRONE_SERVER
          value: drone-service:9000
        # issue: https://github.com/drone/drone/issues/2048
        - name: DOCKER_API_VERSION
          value: "1.24"
        - name: DRONE_SECRET
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: server.secret
      volumes:
        - name: docker-socket
          hostPath:
            path: /var/run/docker.sock
```
所有都部署到devops命名空间下就可以.