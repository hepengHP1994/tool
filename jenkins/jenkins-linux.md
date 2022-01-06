# jenkins安装说明


### 1.安装好java 环境

### 2.安装jenkins
```
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.277.4-1.1.noarch.rpm
echo '开始安装jenkins'
yum -y localinstall jenkins-2.277.4-1.1.noarch.rpm || (
    echo 'jenkins安装失败'
    exit 1
)
#安装字体，不然Jenkins启动后会报错
yum install fontconfig

```

###  3. /etc/sysconfig/jenkins 配置jenkis的端口
```
安装完成后，不要忙着启动，需要修改几个配置文件
/etc/sysconfig/jenkins 修改 JENKINS_PORT=“8080”，将默认的8080端口修改为没有被占用的，因为8080一般都会被占用.
JENKINS_USER=“jenkins”，建议修改为root，并且赋权
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

### 4.编辑/etc/init.d/jenkins文件, 指定位置添加Java的环境变量地址
```
/etc/init.d/jenkins 在candidates=" "里面加上Java的安装路径

candidates="
#Java的安装路径
/usr/local/java/jdk-11.0.9/bin/java
"
```

修改完成后 systemctl daemon-reload 更新一下配置




### 5.启动jenkins
```
service jenkins start/stop/restart  或者  systemctl start jenkins
```


### 6.打开jenkins
```
初始密码在：/var/lib/jenkins/secrets/initialAdminPassword
选择“Install suggested plugins”安装默认的插件，下面Jenkins就会自己去下载相关的插件进行安装。
```


### 7.安装插件 在Manage Jenkins > Manage Plugins
```
1.Docker plugin
2.docker-build-step
3.ssh plugin
4.Publish Over SSH
5.git
6.Maven Integration
```
### 8.配置maven 与jdk    在Manage Jenkins > Global Tool Configuration
```
点击新增jdk 出现 别名和 JAVA_HOME 两栏
别名谁便填，JAVA_HOME 填写服务器上java的路径

点击新增maven 出现 别名和 MAVEN_HOME 两栏
别名谁便填，MAVEN_HOME 填写服务器上maven的路径

```
