#linux 安装python

### 1、依赖包安装
    yum -y install gcc-c++ zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel mesa-libGL.x86_64

### 2、下载包：
    wget https://www.python.org/ftp/python/3.7.11/Python-3.7.11.tgz

### 3、解压：
    tar -zxvf Python-3.7.11.tgz

### 4、安装：
``` linux
    cd Python-3.7.11

    ./configure --prefix=/usr/local/python/python38 --enable-shared && make && make install
```
    
### 5、配置
``` linux
    # 把python的动态库复制到 /usr/lib64
    cp /usr/local/python38/lib/libpython3.8.so.1.0  /usr/lib64   

    # 添加软链接
    ln -s /usr/local/python/python37/bin/python3.7 /usr/bin/python37
    ln -s /usr/locall/python/python37/bin/pip3.7 /usr/bin/pip37
```

