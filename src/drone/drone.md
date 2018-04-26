#### Drone 集成到Gitlab
Drone 支持 GitLab，使用下面的环境变量来配置使用 GitLab。
```docker-compose
version: '2'

services:
  drone-server:
    image: drone/drone:0.7
    ports:
      - 80:8000
    volumes:
      - /var/lib/drone:/var/lib/drone/
    restart: always
    environment:
      - DRONE_OPEN=true
      - DRONE_GITLAB=true
      - DRONE_GITLAB_CLIENT={DRONE_GITLAB_CLIENT}
      - DRONE_GITLAB_SECRET={DRONE_GITLAB_SECRET}
      - DRONE_GITLAB_URL=http://gitlab.test.com
      - DRONE_SECRET=${DRONE_SECRET}
  drone-agent:
    image: drone/drone:0.7
    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_SERVER=ws://drone-server:8000/ws/broker
      - DRONE_SECRET=${DRONE_SECRET}
```
其中的通信密钥相当于 drone 的密码，最好为一个长传的随机字串防止被破解。可以在命令行中使用一下内容生成：
```bash
 > LC_ALL=C </dev/urandom tr -dc A-Za-z0-9 | head -c 65 && echo
```


#### 配置
下面是所有的配置选项。一般来说，使用默认配置可以满足绝大部分的安装需求：

* DRONE_GITLAB=true
  true 使用 GitLab
* DRONE_GITLAB_URL=https://gitlab.com
  GitLab Server 地址
* DRONE_GITLAB_CLIENT
  GitLab oauth2 client id
* DRONE_GITLAB_SECRET
  GitLab oauth2 client secret
* DRONE_GITLAB_GIT_USERNAME
  可选，使用单一用户来克隆所有仓库，这个用户的用户名
* DRONE_GITLAB_GIT_PASSWORD
  可选，使用单一用户来克隆所有仓库，这个用户的密码
* DRONE_GITLAB_SKIP_VERIFY=false
  设置 true 来取消 SSL 检查
* DRONE_GITLAB_PRIVATE_MODE=false
  如果 GitLab 以 private 私有模式运行，应设置为 true

#### 注册应用程序
在 GitLab 上注册一个应用，并生成一个客户端和密钥。访问账户设置（account settings）页面，选择 Applications 页面，点击 New Application 。

使用下面的认证回调 URL（Authorization callback URL），请修改域名为自定义域名： http://drone.mycompany.com/authorize


#### 启动服务

配置文件完成之后，我们就可以使用以下命令启动服务了：

```bash
 > docker-compose  up
```

docker-compose 会自动帮我们去下载镜像并根据配置初始化容器。一切就绪之后，我们使用 <host>:3800 就可以访问到 drone 了。

#### 配置 Nginx

我们配置下 Nginx 做下反向代理就可以了，以下是具体的 nginx 配置文件示例：
```markdown
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen 80;
    server_name ci.eming.li;
    set $drone_port 3800;
    location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:$drone_port;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_buffering off;
        chunked_transfer_encoding off;
    }
    location ~* /ws {
        proxy_pass http://127.0.0.1:$drone_port;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
    }
}

```
修改 server_name 和 $drone_port 为正确的值即可，最后记得别忘了重启 Nginx 服务。
这样我们就能直接使用 http://gitlab.test.com来访问 drone 了。


#### 进程守护

  
