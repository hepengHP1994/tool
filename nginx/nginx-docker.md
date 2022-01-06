# nginx 安装说明

### 1.docker-compose脚本
```
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    # 给容器加上特定权限 防止挂载信息时出出现权限不足
    privileged: true
    ports:
      - 80:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./wwwroot:/usr/share/nginx/wwwroot
```



### 2.创建conf/nginx.conf


```
# 启动进程,通常设置成和 CPU 的数量相等
worker_processes  1;

events {
    # epoll 是多路复用 IO(I/O Multiplexing) 中的一种方式
    # 但是仅用于 linux2.6 以上内核,可以大大提高 nginx 的性能
    use epoll;
    # 单个后台 worker process 进程的最大并发链接数
    worker_connections  1024;
}

http {
    # 设定 mime 类型,类型由 mime.type 文件定义
    include       mime.types;
    default_type  application/octet-stream;

    # sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    # 必须设为 on，如果用来进行下载等应用磁盘 IO 重负载应用，可设置为 off，以平衡磁盘与网络 I/O 处理速度，降低系统的 uptime.
    sendfile        on;

    # 连接超时时间
    keepalive_timeout  65;
    # 设定请求缓冲
    client_header_buffer_size 4k;

    # 配置一个代理即服务器
    upstream nacosService{
      server 服务器ip:8848 weight=10;
    }


    # 配置一个虚拟主机
    server {
	      # 监听的ip和端口，配置 服务器ip:80
        listen      80;
	      # 虚拟主机名称这里配置ip地址
        server_name  服务器ip;
	      # 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
	    # 使用 root 指令指定虚拟主机目录即网页存放目录
	    # 比如访问 http://ip/index.html 将找到 /usr/local/docker/nginx/wwwroot/html80/index.html
	    # 比如访问 http://ip/item/index.html 将找到 /usr/local/docker/nginx/wwwroot/html80/item/index.html
	    proxy_pass http://nacosService;

	    # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }
    }

}


```
