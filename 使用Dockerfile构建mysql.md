[TOC]





## 一、前提

部署了docker环境



## 二、下载基础镜像

```bash
# docker pull 10.10.1.64:180/mysql/mysql:5.6.41
```



## 三、准备目录

```bash

#mkdir -p /data/dataplatform/mysql/data #存放 data
#mkdir -p /data/dataplatform/mysql/config #存放配置文件
[root@bigdata-18 config]# cat my.cnf
[client]
port=3306
default-character-set =utf8
[mysqld]
port=3306
user=mysql
server-id = 111
log_bin=/var/lib/mysql/log
datadir=/var/lib/mysql/data
default-time-zone=system
character-set-server=utf8
default-storage-engine=InnoDB
explicit_defaults_for_timestamp=true

                    
```



## 五、启动容器

```bash
# docker run -p 3306:3306 --name mymysql -v /data/dataplatform/mysql/config/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf  -v /data/dataplatform/mysql/data:/var/lib/mysql/data -v /data/dataplatform/mysql/logs:/var/lib/mysql/log -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6.41  
#启动容器，-d后台运行，-p端口映射，-v容器文件映射到本地，restart是docker启动的时候容器自动重启

# docker ps
CONTAINER ID        IMAGE                                            COMMAND                  CREATED             STATUS              PORTS                                                NAMES
922a3469cc05        mysql:5.6.41                                     "docker-entrypoint.s…"   35 seconds ago      Up 33 seconds       0.0.0.0:3306->3306/tcp                               mymysql 


```



