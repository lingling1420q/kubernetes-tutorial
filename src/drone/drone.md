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
      - DRONE_GITLAB_URL={DRONE_GITLAB_URL}
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



  
