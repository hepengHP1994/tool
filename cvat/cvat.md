# cvat 标注工具安装说明

### 1.下载项目
```
  1.直接去git下载压缩包
  2.通过 git 克隆 git clone https://github.com/opencv/cvat
```


### 2.启动项目
```
  1. 本地主机之外访问CVAT实例，应指定CVAT主机环境变量
     执行 echo "export CVAT_HOST=本机ip" >> /etc/profile && source /etc/profile

  2.执行docker-compose 启动命令

  3. 执行docker exec -it cvat bash -ic 'python3 ~/manage.py createsuperuser' 创建超级管理员账户
  4. 注：docker版本会部署报错 暂时只试了升级到20.10.12版本就ok了


```
