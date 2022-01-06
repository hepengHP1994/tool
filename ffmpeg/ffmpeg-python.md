# 创建ffmpeg + python 的docker 环境

### 1.安装需要的控件
    1.yasm-1.3.0  这个组件呢主要是为ffmpeg编译提供类似加速的功能，安装x264也需要
    2.x264   X264静态库  ffmpeg需要进行264编码 所以要安装
    3.ffmpeg ffmpeg 进行视频推流
    4.python jdk python环境需要 自己去下载 wget https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tgz

下载路径
* 链接：https://pan.baidu.com/s/1v3mhTLLY09aLsqE1boJ6cw 
* 提取码：7q8n

*****

### 2.编写Dockerfile 文件
``` Dockerfile
# 在centos7的基础镜像
FROM centos:7 

# 创建一个文件夹
RUN mkdir -p /home/webapps 

# 把 控件 解压到 指定文件夹中 到时候更具自己的文件的名称进行修改
ADD yasm-1.3.0.tar.gz /home/webapps 
ADD ffmpeg-4.3.2.tar.gz /home/webapps 
ADD x264-master.tar.gz /home/webapps  
ADD Python-3.8.12.tgz /home/webapps



# 更新yum 并且 安装需要的插件
RUN yum -y update && yum -y install gcc gcc- g++ pcre pcre-devel openssl openssl-devel zlib zlib-devel libffi-devel mesa-libGL.x86_64 bzip2-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel -y 

# 安装 python
RUN  cd /home/webapps/Python-3.8.12 && ./configure --prefix=/usr/local/python38 --enable-shared && make && make install 
# 给python添加软连接
RUN  ln -s /usr/local/python38/bin/python3.8 /usr/local/bin/python38 && ln -s /usr/local/python/bin/pip3 /usr/local/bin/pip3 

# 把python3.8的动态库复制到 /usr/lib64
RUN  cp /usr/local/python38/lib/libpython3.8.so.1.0  /usr/lib64   

# 进入 yasm-1.3.0目录 编译yasm 并且安装
RUN  cd /home/webapps/yasm-1.3.0 && ./configure && make && make install

# 进入 x264-master目录 编译x264 解码器 并且安装 注：类似--enable-shared 这些参数不懂的就百度吧
RUN cd /home/webapps/x264-master && ./configure --enable-shared --enable-static --disable-asm  && make && make install

# 进入 ffmpeg目录 编译ffmpeg 并且安装 
# --enable-libx264  这个是加载x264编码格式 需要完成上一步
# --prefix 指定安装到那个目录下
RUN cd /home/webapps/ffmpeg-4.3.2 && ./configure --enable-libx264 --enable-gpl  --prefix=/usr/local/ffmpeg && make && make install

# 配置ffmpeg 最后 使配置生效
RUN  echo "/usr/local/ffmpeg/lib" > /etc/ld.so.conf.d/ffmpeg.conf &&  echo "/usr/local/lib" >> /etc/ld.so.conf.d/ffmpeg.conf && ldconfig

# 清除yum的全部缓存 删除安装完的控件目录
RUN  yum clean all  &&  cd /home/webapps &&  rm -rf  Python-3.8.12 &&   rm -rf  yasm-1.3.0 &&  rm -rf  x264-master &&  rm -rf  ffmpeg-4.3.2

ENV PATH=/usr/local/ffmpeg/bin:$PATH
CMD ["/bin/bash"]  
```

*****


1、./configure 是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本。

2、make 是用来编译的，它从Makefile中读取指令，然后编译。

3、make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
    
            



        


 
    

