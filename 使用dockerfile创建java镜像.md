## 一、目的	

使用dockerfile方法创建一个基于centos7和jdk1.8的镜像



## 二、编译dockerfile

2.1 准备dockerfile文件

```bash
# cat dockerfile 
FROM 10.10.1.64:180/centos/centos:7.3.1611
MAINTAINER MYSH
ADD jdk-8u151-linux-x64.tar.gz /usr/local #
ENV JAVA_HOME /usr/local/jdk1.8.0_151
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:$JAVA_HOME/bin
```

#ADD将主机构建环境（上下文）目录中的文件和目录、以及一个URL标记的文件 拷贝到镜像中。格式是： ADD  源路径  目标路径。如果源文件是个归档文件（压缩文件），则docker会自动帮解压



2.2 上传jdk文件



2.3 使用dockerfile创建镜像

```bash
# docker build -t jdk-8u151:20190919 . -f dockerfile 
```



2.4 查看创建的镜像

```bash
# docker images
REPOSITORY                                 TAG                        IMAGE ID            CREATED             SIZE
jdk-8u151                                  20190919                   2d72d9d08376        27 seconds ago      576MB
```



## 三、运行

3.1 运行创建的镜像

```bash
#docker run -d -it  --name jdk_prod --restart=always jdk-8u151:20190919
#jdk_prod是自定义容器名，--restart=always指宿主机重启后容器随机自启动
701bc3d08b7cebd53af8e00695e68d7e88b8db9cbcdf47b5ef23e1e5d30b391a
```



3.2 验证

```bash
# docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS                         PORTS               NAMES
955664ac6e1e        jdk-8u151:20190919   "/bin/bash"         2 minutes ago       Up 56 seconds                                      jdk_prod

# docker exec -it 701b sh
sh-4.2# java -version
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
```

