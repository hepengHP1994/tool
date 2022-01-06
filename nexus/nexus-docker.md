Copyright (c) 2018 Copyright Holder All Rights Reserved.# nexus 安装说明

### 1.docker-compose脚本
```
version: '3.1'
services:
  #maven仓库
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    # 给容器加上特定权限 防止挂载信息时出出现权限不足
    privileged: true
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
```


### 2.默认密码在 data/admin.password




### 3.配置maven从私库获取
```
maven 默认安装在/usr/local/java/apache-maven-3.6.3目录下

配置文件在/usr/local/java/apache-maven-3.6.3/conf/settings.xml

1.配置server账户 在servers标签里配置
  <server>
      <id>maven-xxx</id>
      <username>XXXXX</username>
      <password>xxxxxxx</password>
  </server>

2.配置profile 在profiles标签里配置
  <profile>
    <id>dev</id>
    <repositories>
    <repository>
      <id>maven-public</id>
      <url>http://私库IP:8081/repository/maven-public/</url>
    </repository>
    <!-- 私服发布仓库，即私服正式jar仓库 -->
    <repository>
      <id>maven-releases</id>
      <url>http://私库IP:8081/repository/maven-releases/</url>
    </repository>
    <!-- 私服快照仓库，即私服临时jar仓库 -->
    <repository>
      <id>maven-snapshots</id>
      <url>http://私库IP:8081/repository/maven-snapshots/</url>
    </repository>
    </repositories>
    <!-- 私服插件仓库，一般插件都是从外网仓库下载，可以不用配置 -->
    <pluginRepositories>
      <pluginRepository>
        <id>maven-releases</id>
        <url>http://私库IP:8081/repository/maven-releases/</url>
      </pluginRepository>
      <pluginRepository>
        <id>maven-snapshots</id>
        <url>http://私库IP:8081/repository/maven-snapshots/</url>
      </pluginRepository>
      <pluginRepository>
        <id>maven-public</id>
        <url>http://私库IP:8081/repository/maven-public/</url>
      </pluginRepository>
    </pluginRepositories>
  </profile>


3.在激活一下
<activeProfiles>
    <activeProfile>dev</activeProfile>
</activeProfiles>

```
