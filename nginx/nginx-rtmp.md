# 创建nginx推流服务

### 1.安装需要的控件
    1.nginx 
    2.nginx-http-flv-module 推流模块

下载路径
* 链接：https://pan.baidu.com/s/1pvzV6T-CYVLBYY7Yl-2Jng 
* 提取码：t62w


*****

### 2.安装脚本
``` sh
#! /bin/bash
# 开放80端口 
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 开放1935端口
firewall-cmd --zone=public --add-port=1935/tcp --permanent

# 配置立即生效
firewall-cmd --reload

# 安装插件
yum -y install gcc gcc- g++ pcre pcre-devel openssl openssl-devel zlib zlib-devel -y

# 解压nginx 
tar -zxvf nginx/nginx-1.21.0.tar.gz -C nginx/
rm -f nginx/nginx-1.21.0.tar.gz

# 解压nginx 推流模块
tar -zxvf nginx/nginx-http-flv-module-master.tar.gz -C nginx/
rm -f nginx/nginx-http-flv-module-master.tar.gz

# 进入 nginx 目录
cd nginx/nginx-1.21.0

# 安装nginx 并且加载 nginx 推流模块
./configure --prefix=/usr/local/nginx  --add-module=../nginx-http-flv-module-master  --with-http_ssl_module && make && make install


# 复制准备好nginx.conf 配置文件 到安装nginx 的目录下
\cp -rf ../nginx.conf /usr/local/nginx/conf/

# 启动nginx 
/usr/local/nginx/sbin/nginx

# 添加到自启中
echo "/usr/local/nginx/sbin/nginx" >> /etc/rc.local
chmod 755  /etc/rc.local

```