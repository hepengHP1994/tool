# docker 镜像仓库 安装说明


### 1.docker-compose脚本
```
version: '3.1'
services:
  registry:
    #docker镜像仓库
    image: registry
    restart: always
    container_name: registry
    # 给容器加上特定权限 防止挂载信息时出出现权限不足
    privileged: true
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
      - /usr/local/docker/registry/conf:/etc/docker/registry
  #docker镜像仓库页面
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    restart: always
    # 给容器加上特定权限 防止挂载信息时出出现权限不足
    privileged: true
    ports:
      - 5001:80
    volumes:
      - ./certs/frontend.crt:/etc/apache2/server.crt:ro
      - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=仓库ip
      - ENV_DOCKER_REGISTRY_PORT=仓库端口
```

### 2.创建conf/config.yml
```
version: 0.1
log:
  fields:
    service: registry
storage:
  #开启删除镜像功能
  delete:
     enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3

```


### 3.推镜像到仓库
  1.需要在docker 中配置私库的ip与端口
    打开docker配置文件/etc/docker/daemon.json  添加
    "insecure-registries": [
    "docker私库ip:docker私库端口","docker私库ip:docker私库端口"
    ]
    docker私库可配置多个以逗号分割
  2.推镜像命令
    1.给镜像打标签   docker tag 镜像名:版本号 私库ip:端口/需要的镜像名:版本号
    2.推镜像     docker push 私库ip:端口/需要的镜像名:版本号

  3.拉私库镜像
    docker pull 私库ip:端口/镜像名:版本号
