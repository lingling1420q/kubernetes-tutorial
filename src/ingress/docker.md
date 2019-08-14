
#### Dockerfile
```dockerfile
#母镜像
FROM golang:latest as build
#维护者信息
MAINTAINER keke

ENV GOPROXY https://goproxy.io/
# go module开启
ENV GO111MODULE on

WORKDIR /go/cache

# 添加go mod
ADD go.mod .
ADD go.sum .

# 构建缓存包含了该项所有的依赖起到加速构建的作用
RUN go mod download

#工作目录
WORKDIR /go/release

#将文件复制到镜像中
ADD . .

# ldflags中-s: 省略符号表和调试信息,-w: 省略DWARF符号表
RUN GOOS=linux CGO_ENABLED=0 go build -ldflags="-s -w" -installsuffix cgo -o quest main.go

# scratch空的基础镜像，最小的基础镜像
# busybox带一些常用的工具，方便调试， 以及它的一些扩展busybox:glibc
# alpine另一个常用的基础镜像，带包管理功能，方便下载其它依赖的包
FROM scratch as prod

# 配置镜像的时间区
COPY --from=build /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 配置镜像的证书
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
# 将构建的可执行文件复制到新镜像中
COPY --from=build /go/release/quest /
# 将配置赋值到新镜像
COPY --from=build /go/release/conf/conf.yaml /

EXPOSE 10835

CMD ["/quest"]

```

我这个项目有一些外部依赖，在本地开发的时候都已调整好，并且编译通过，在本地开发环境已经生成了两个文件go.mod、go.sum.

在dockerfile的第一步骤中，先启动module模式，且配置代理，因为有些墙外的包服务没有梯子的情况下也是无法下载回来的，这里的代理域名是我自己的，有需要的也可以用。
指令RUN go mod download执行的时候，会构建一层缓存，包含了该项所有的依赖。

之后再次提交的代码中，若是go.mod、go.sum没有变化，就会直接使用该缓存，起到加速构建的作用，也不用重复的去外网下载依赖了。若是这两个文件发生了变化，就会重新构建这个缓存分层。
