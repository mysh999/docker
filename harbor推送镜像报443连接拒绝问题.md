## 一、问题描述

部署了一套harbor镜像仓库，对docker镜像进行push的时候，报以下提示：

```bash
# docker push 10.10.2.34/library/jenkins-slave-jdk:1.8       
The push refers to repository [10.10.2.34/library/jenkins-slave-jdk]
Get https://10.10.2.34/v2/: dial tcp 10.10.2.34:443: connect: connection refused
```





原因：

Docker Registry 交互默认使用的是 HTTPS，但是搭建私有镜像默认使用的是 HTTP 服务，所以与私有镜像交互时出现以下错误。







## 二、解决办法

```bash
# vi /usr/lib/systemd/system/docker.service 
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 修改成
ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry 10.10.2.34 --containerd=/run/containerd/containerd.sock 

# 重新加载和启动
# systemctl daemon-reload
# systemctl restart docker

# docker login 10.10.2.34
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
# docker push 10.10.2.34/library/jenkins-slave-jdk:1.8
```

