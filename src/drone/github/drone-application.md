#### Drone 使用

.drone.yml

````yaml
workspace:
  base: /go
  # 指定git clone到的地方, 应该放在gopath下, 才能正常编译
  path: src/spike

pipeline:
  build-develop:
    image: golang:1.9
    commands:
      - pwd
      - go version
      # 如果要在alpine上运行编译后文件则必须添加这些参数, 参见http://docs.drone.io/creating-custom-plugins-golang/
      - GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o app
      - cp -f config.test.yml config.yml
    when:
      branch: develop

  # 需要在drone填写secrets: docker_username, docker_password
  publish-develop:
    image: plugins/docker
    mirror: https://docker.mirrors.ustc.edu.cn
    registry: registry-internal.cn-hangzhou.aliyuncs.com # 仓库
    repo: registry-internal.cn-hangzhou.aliyuncs.com/zhuzi/drone-test # docker仓库地址
    secrets: [ docker_username, docker_password ]
    tags:
      - test
    when:
      branch: develop

  # 插件会干两件事情, 1. upgrade 2. finish upgrade
  # 注意如果这个服务不在Active状态就不能正常升级, 需要登录rancher修改状态, 正常情况下不会发生这个错误.
  rancher-develop:
    image: peloton/drone-rancher
    url: http://rancher.bysir.store/v1
    access_key: ""
    secret_key: ""
    service: app/drone-test
    # 为了使rancher能拉取到私有镜像, 需要在rancher控制面板"基础架构->镜像库"添加这个私有镜像库
    docker_image: registry-internal.cn-hangzhou.aliyuncs.com/zhuzi/drone-test:test
    start_first: true # 先启动新服务, 后停止原服务. 如果为false则先关闭原服务再启动
    confirm: true
    timeout: 100 # 如果rancher没在这个时间内升级成功则报错, 服务大小等差异会导致升级时间不一样, 可根据自己业务修改超时时间.
    when:
      branch: develop

````

* workspace

其中workspace指定pipeline的工作目录, 上例中我们会在build中pwd看到当前目录是/go/src/spike, 
为什么我们需要指定到/go目录下, 因为在golang:1.9的镜像中, go_path就是/go, 我们要go build当然要在go_path下执行.
