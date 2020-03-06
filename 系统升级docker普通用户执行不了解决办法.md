- 问题描述：

docker服务器在系统做了yum update升级后，发现普通用户执行docker命令报



```bash
$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```





- 解决办法：

```bash
$ sudo chmod 666 /var/run/docker.sock 

$ docker ps
CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                    NAMES
f81aab40931d        tomcat:8.5.23                              "/bin/sh -c './sta..."   45 hours ago        Up 2 minutes        0.0.0.0:8003->8080/tcp   opertaion-rest
```

