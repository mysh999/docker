## 一、问题描述

docker登录harbor时，提示如下：

```bash
#  docker login 192.168.0.211       
Username: mysh
Password: 
Error response from daemon: Get https://192.168.0.211/v2/: dial tcp 192.168.0.211:443: connect: connection refused
```





## 二、解决办法

2.1、删除daemon.json配置文件

```bash
# rm -f /etc/docker/daemon.json
```



2.2、配置insecure

```bash
# vi /usr/lib/systemd/system/docker.service 
# 新增insecure-registry项
ExecStart=/usr/bin/dockerd --insecure-registry=192.168.0.211   
```



重启服务

```bash
# systemctl daemon-reload       
# systemctl restart docker    
```



2.3、登录测试

```bash
#  docker login 192.168.0.211
Username: mysh
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


# cat  /root/.docker/config.json
{
        "auths": {
                "192.168.0.204:38880": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                },
                "192.168.0.211": {
                        "auth": "bXlzaDpIYXJib3IxMjM0NQ=="
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/19.03.9 (linux)"
        }


# docker tag emqx/emqx:4.3.11 192.168.0.211/dev/emqx/emqx:4.3.11
# docker push 192.168.0.211/dev/emqx/emqx:4.3.11
The push refers to repository [192.168.0.211/dev/emqx/emqx]
2ab8eaead759: Pushed 
f79cf0b71d9f: Pushed 
51b6c86045dc: Pushed 
74a145db9522: Pushed 
ebb6c3a78ef2: Pushed 
d6960d2ef6f2: Pushed 
eb4bde6b29a6: Pushed 
4.3.11: digest: sha256:3d0dc10346d561ee30353b97f1cf4aa0ba72f74c1d91bbf170bbc39297b208aa size: 1786
```

