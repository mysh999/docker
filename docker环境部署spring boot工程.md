## 一、目标

使用docker容器部署一套java spring boot工程



## 二、准备条件

因为spring boot后台有调用mysql、redis、zookeeper，因此需要部署这些环境，架构如下：

- mysql：单机架构
- redis：单机架构
- zookeeper：集群架构



## 三、docker环境准备

3.1、安装docker

```bash
# yum -y remove docker*
# yum -y install yum-utils device-mapper-persistent-data lvm2
# yum -y install docker-ce
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum -y install docker-ce
# systemctl start docker
# systemctl enable docker
# docker ps

# docker -v
Docker version 19.03.5, build 633a0ea
```



3.2、安装docker-compose

下载docker-compose二进制文件复制到/usr/bin目录下，且赋予执行权限

```bash
#查看版本
# docker-compose -v
docker-compose version 1.25.1, build a82fef07
```



3.3、配置防火墙策略

```bash
# systemctl enable firewalld && systemctl start firewalld
# firewall-cmd --add-rich-rule="rule family="ipv4" source address="0.0.0.0/0" accept" --permanent
# iptables-save
```



## 四、配置mysql

4.1、准备mysql配置文件

```bash
# cat my.cnf 
[mysqld]
## 设置server_id，一般设置为IP，注意要唯一
server_id=100  
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql  
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=mysql-bin  
## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M  
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed  
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7  
log_error=/var/log/mysql/mysql_error.log
general_log_file=/var/log/mysql/mysql.log
general_log=1
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql_slow.log
long_query_time=2
log_queries_not_using_indexes=1

```



4.2、准备Dockerfile文件

```bash
# cat Dockerfile_mysql 
FROM mysql:5.6
MAINTAINER mysh
ADD ./my.cnf /etc/mysql/my.cnf  
```



4.3、准备mysql初始化脚本

```bash
# ls -lt ./mysql_init/
total 108
-rw-r--r-- 1 root root 41295 Dec 23 15:44 ubxdb.sql
-rw-r--r-- 1 root root 57882 Dec 23 15:42 abiclouddb.sql
-rw-r--r-- 1 root root  1523 Dec 23 15:41 cbiatrisdb.sql
```



4.4、准备docker-compose配置文件

```bash
# cat mysql.yml 
version: '3'                            #版本3语法
services:
  mysql-5.6:                            #定义服务名
   build:
      context: ./
      dockerfile: ./Dockerfile_mysql
   container_name: mysql_prod           #定义容器名称
   hostname: mysql_prod
   environment:                         #设置环境变量
    MYSQL_ROOT_PASSWORD: 123456         #root初始密码
    MYSQL_DATABASE: proddb              #新建库
    MYSQL_ROOT_HOST: '%'                #允许来自运行容器的主机连接
    TZ: Asia/Shanghai                   #设置时区
   ports:                               #端口映射
   - "3306:3306"
   volumes:                             #卷映射
   - ./mysql_data:/var/lib/mysql
   - ./mysql_logs:/var/log/mysql
   - ./mysql_init:/docker-entrypoint-initdb.d   #初始化sql
   - /usr/share/zoneinfo/Asia/Chongqing:/etc/localtime:ro
   restart: always                      #当docker重启时容器自动启动
   networks:
     - default


networks:                              #自定义网络
  default:
    external:
      name: spring            
```



4.5、创建docker网络

```bash
# docker network create spring
```



4.6、启动mysql容器

```bash
# docker-compose -f mysql.yml up -d --build
```





## 五、配置redis

5.1、准备redis配置文件

```bash
# cat redis.conf 
bind  0.0.0.0
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis/redis.pid
loglevel notice
logfile /var/log/redis/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /data
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
requirepass password123
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes

rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG ""
rename-command EVAL ""
```



5.2、准备redis docker-compose文件

```bash
# cat redis.yml 
version: '2'
services:

  #redis容器
  redis:
    #定义主机名
    hostname: myredis
    #使用的镜像
    image: redis:5.0.2
    #容器的映射端口
    ports:
      - 6379:6379
    restart: always
    #定义挂载点
    volumes:
      - ./redis_data:/data
      - ./redis_logs:/var/log/redis
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command:
         - /bin/bash
         - -c
         - chown -R root /var/log/redis&&chgrp -R root /var/log/redis&&redis-server /usr/local/etc/redis/redis.conf #赋予权限
    #环境变量
    environment:
      - TZ=Asia/Shanghai
      - LANG=en_US.UTF-8
    networks:
      - default


networks:
  default:
    external:
      name: spring
```



5.3、执行build启动容器

```bash
# docker-compose -f redis.yml up -d --build
```





## 六、配置zookeeper

6.1、配置zk  docker-compose文件

```bash
# cat zookeeper.yml 
version: '2'
services:
    zoo1:
        image: zookeeper:3.5.5
        restart: always
        container_name: zoo1
        hostname: zoo1
        ports:
            - "2181:2181"
        volumes:
        - "./zoo1/data:/data"
        - "./zoo1/datalog:/datalog"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
        networks:
          - default

    zoo2:
        image: zookeeper:3.5.5
        restart: always
        container_name: zoo2
        hostname: zoo2
        ports:
            - "2182:2181"
        volumes:
        - "./zoo2/data:/data"
        - "./zoo2/datalog:/datalog"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
        networks:
          - default

    zoo3:
        image: zookeeper:3.5.5
        restart: always
        container_name: zoo3
        hostname: zoo3
        ports:
            - "2183:2181"
        volumes:
        - "./zoo3/data:/data"
        - "./zoo3/datalog:/datalog"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
        networks:
          - default


networks:
  default:
    external:
      name: spring
```



6.2、启动docker

```bash
# docker-compose -f zookeeper.yml up -d --build
```



## 七、配置spring boot工程

这里以ubtechinc-abi-micro-service-1.0.0.jar工程为例



7.1、修改工程配置内容

进入 ubtechinc-abi-micro-service-1.0.0.jar\BOOT-INF\classes 目录

修改application-dev.yml文件

![360截图181412196410775.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gaqaz0l3jij30ma0s4aaw.jpg)



7.2、配置Dockerfile文件

```bash
# cat Dockerfile 
FROM java:1.8.0_151

ARG JARS_NAME
ARG RUNS_OPTS
ARG RUNS_PROFILES
ENV RUNS $RUNS_OPTS
ENV PROFILES $RUNS_PROFILES            # 指定profile启动
ADD $JARS_NAME /app.jar                # jar名称
RUN bash -c 'touch /app.jar'

ENTRYPOINT [ "sh", "-c", "java $RUNS   -Djava.security.egd=file:/dev/./urandom -jar /app.jar  --spring.profiles.active=$PROFILES" ]
```



7.3、配置工程docker-compose文件

```bash
# cat ubtechinc-abi-micro-service.yml 
version: '2.0'
services:
  ubtechinc-abi-micro-service:                                      # 服务名
    build:                                                          # build相关信息
     context: ./                                                   # build路径
      #dockerfile: ubtechinc.dockerfile                              # dockerfile
     args:
      JARS_NAME: ubtechinc-abi-micro-service-1.0.0.jar         # jar名称
      #JARS_NAME: ubtechinc-abi-micro-service-1.0.0.jar         # jar名称
      RUNS_OPTS: -Xmx256M -Xms256M                                   #启动参数
      RUNS_PROFILES: dev                                            # 指定profile启动
    image: ubtechinc-abi-micro-service:1104                       # 镜像版本号
    container_name: ubtechinc-abi-micro-service                                  # 容器名称
    restart: always
    volumes:
      - ./logs/:/logs
    ports:
      - 8100:8080
    external_links:
      - mysql_prod
      - java_redis_1
      - zoo1
      - zoo2
      - zoo3
    networks:
      - default


networks:
  default:
    external:
      name: spring
```



7.4、启动容器

```bash
# docker-compose -f ubtechinc-abi-micro-service.yml up -d --build
```



7.5、查看上下文

进入 ubtechinc-abi-micro-service-1.0.0.jar\BOOT-INF\classes\application.yml文件 

![360截图17970210107125133.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gaqb8zrbrgj30du0dtdfz.jpg)



7.6、查看接口效果

浏览器输入

http://192.168.255.102:8100/v1/abi/swagger-ui.html

![360截图17860602226354.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gaqba3i4p7j319r0cc753.jpg)



