[TOC]



## 一、安装docker和docker-compose

OS:Centos7.5



- 安装docker

```
yum -y remove docker*
yum -y install yum-utils device-mapper-persistent-data lvm2
yum -y install docker-ce
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl start docker
systemctl enable docker
docker ps
```

- 安装docker-compose

```
yum -y install epel-release
yum -y install python-pip
pip install docker-compose
pip install --upgrade pip
pip install docker-compose --ignore-installed requests

#  docker-compose -version
docker-compose version 1.24.1, build 4667896
```



## 二、安装图形界面harbor（包含私有仓库）

1、配置docker

```bash
# vi /usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=192.168.255.101   #追加--insecure-registry=192.168.255.101
```



2、下载安装Harbor

https://github.com/vmware/harbor/releases

选择离线包下载

```bash
# wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.2.tgz
# tar -zxvf harbor-offline-installer-v1.8.2.tgz 
# cd harbor/
# vim harbor.yml  //修改如下项，其他默认
hostname: 192.168.255.101  #本机IP
harbor_admin_password: 123456  #admin密码

# sh install.sh 
如果报
ERROR: for registry  Cannot create container for service registry: Conflict. The container name "/registry" is already in use by container "c8e95728f1af8fe1fd68b09d087e6b1f9925063784f565fc236e668651a2558c". You have to remove (or rename) that container to be able to reuse that name.

解决办法：
将之前的/registry容器删掉 
# docker stop c8e95728f1af
c8e95728f1af
# docker rm c8e95728f1af
c8e95728f1af

```



3、web登陆

http://192.168.255.101



![1565771546825](C:\Users\ubt\AppData\Roaming\Typora\typora-user-images\1565771546825.png)



4、新建项目

![1565771610232](C:\Users\ubt\AppData\Roaming\Typora\typora-user-images\1565771610232.png)



5、本地登录 并且上传镜像

```bash
# docker login 192.168.255.101
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


#下载
# docker pull  centos:latest

#制作tag
# docker tag centos:latest 192.168.255.101/prod/centos:latest  #tag 名称= 仓库地址/项目名称/镜像名称:标记(版本号)

#上传
# docker push 192.168.255.101/prod/centos:latest
```

界面结果：

![1565774491109](C:\Users\ubt\AppData\Roaming\Typora\typora-user-images\1565774491109.png)



6、下载镜像

```bash
# docker pull 192.168.255.101/prod/centos:latest
```



## 三、新服下载上传

1、安装docker

参考上面过程，略



2、配置http免密

```bash
# vi /usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=192.168.255.101   #追加--insecure-registry=192.168.255.101
```

3、重启docker服务

```bash
[root@zk ~]# systemctl daemon-reload
[root@zk ~]# systemctl restart docker
```

4、下载

```bash
[root@zk ~]# docker pull 192.168.255.101/prod/centos:latest
latest: Pulling from prod/centos
8ba884070f61: Pull complete 
Digest: sha256:ca58fe458b8d94bc6e3072f1cfbd334855858e05e1fd633aa07cf7f82b048e66
Status: Downloaded newer image for 192.168.255.101/prod/centos:latest
192.168.255.101/prod/centos:latest
```



5、上传

```bash
[root@zk ~]# docker login 192.168.255.101
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@zk ~]# docker pull hello-world
[root@zk ~]# docker tag hello-world:latest 192.168.255.101/library/hello-world:latest
[root@zk ~]# docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
192.168.255.101/prod/centos           latest              9f38484d220f        5 months ago        202MB
192.168.255.101/library/hello-world   latest              fce289e99eb9        7 months ago        1.84kB
hello-world                           latest              fce289e99eb9        7 months ago        1.84kB
[root@zk ~]#  docker push 192.168.255.101/library/hello-world:latest
The push refers to repository [192.168.255.101/library/hello-world]
af0b15c8625b: Pushed 
latest: digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a size: 524
```



