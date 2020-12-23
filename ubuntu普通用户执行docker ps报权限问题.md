## 一、问题描述

ubuntu 普通用户执行docker ps报以下提示：

```bash
$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```





## 二、原因

 docker进程使用 Unix Socket 而不是 TCP 端口。而默认情况下，Unix socket 属于 root 用户，因此需要 **root权限** 才能访问



## 三、解决办法

```bash
#新建docker 组，有就略过
$ sudo groupadd docker 

#将当前用户添加到组
$ sudo gpasswd -a $USER docker
Adding user test to group docker

#更新
$ newgrp docker

#执行，不再报错
$ docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b7f0
 Built:             Wed Mar 11 01:25:58 2020
 OS/Arch:           linux/amd64
 Experimental:      false
```

