[TOC]





## 一、前提

部署了docker环境



## 二、下载基础镜像

```bash
#docker pull centos
```



## 三、准备dockerfile文件

```bash
#mkdir -p /data/dataplatform/nginx
#mkdir -p /data/dataplatform/nginx/data #存放nginx data
#mkdir -p /data/dataplatform/nginx/config #存放config文件

# cat /data/dataplatform/nginx/config/run.sh #容器启动脚本
#!/bin/bash
/usr/local/nginx/sbin/nginx



# cat /data/dataplatform/nginx/Dockerfile
FROM centos  #写在第一条，基础镜像
MAINTAINER ubt #作者
RUN yum install -y wget proc-devel net-tools gcc zlib zlib-devel make openssl-devel
#在当前镜像的顶层执行任何命令

RUN wget http://nginx.org/download/nginx-1.9.7.tar.gz
RUN tar zxvf nginx-1.9.7.tar.gz
WORKDIR nginx-1.9.7 #设置执行命令的工作目录
RUN ./configure --prefix=/usr/local/nginx && make && make install
EXPOSE 80  #容器在运行时要监听的端口                      
RUN echo "daemon off;">>/usr/local/nginx/conf/nginx.conf      
ADD ./config/run.sh /run.sh    #将文件<src>拷贝到container的文件系统对应的路径<dest>下    
RUN chmod 755 /run.sh
CMD ["/run.sh"] #一个Dockerfile里只能有一个CMD，如果有多个，只有最后一个生效.CMD与RUN的区别在于，RUN是在build成镜像时就运行的，CMD会在每次启动容器的时候运行，而RUN只在创建镜像时执行一次，固化在image中
```



## 四、利用dockerfile生成镜像

```bash
# docker build -t nginx:new .     #生成


# docker images   #查看image
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
nginx                                   new                 7028d6d7c799        6 minutes ago       414MB

centos                                  latest              9f38484d220f        5 months ago        202MB
```



## 五、启动容器

```bash
#docker run -d -p 8888:80 -v /data/dataplatform/nginx/data:/usr/local/nginx/html --restart=always nginx:new   #启动容器，-d后台运行，-p端口映射，-v本地文件映射到容器中，restart是docker启动的时候容器自动重启

# docker ps
CONTAINER ID        IMAGE                                            COMMAND                  CREATED              STATUS              PORTS                                                NAMES
faf418f3dd16        nginx:new                                        "/run.sh"                About a minute ago   Up About a minute   443/tcp, 0.0.0.0:8888->80/tcp   


```



