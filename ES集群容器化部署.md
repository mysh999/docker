## 一、需求

4台服务器做master，使用容器技术实现ES集群

10.10.2.12

10.10.2.13

10.10.2.14

10.10.2.15

网络使用host方式



## 二、容器环境准备

过程略



## 三、初始化环境

1、防火墙放行

2、关闭selinux

3、打开ulimit限制

4、配置sysctl

```bash
#vi /etc/sysctl.conf

vm.max_map_count=262144
```

过程略



## 四、配置步骤

​			4个节点都要配置

4.1、准备配置文件



```bash
# cat conf/elasticsearch.yml 
cluster.name: ubtech_es
node.name: es-1
bootstrap.system_call_filter: false
network.host: 10.10.2.12   #该IP根据节点做替换
http.port: 9200
discovery.zen.ping.unicast.hosts: ["10.10.2.12","10.10.2.13","10.10.2.14","10.10.2.15"]        #所有节点IP
discovery.zen.minimum_master_nodes: 2
http.cors.enabled: true
http.cors.allow-origin: "*"
path.repo: ["/usr/share/elasticsearch/snapshot"]
```



4.2 配置docker文件

```bash
# cat docker-compose.yml 
version: '2'
services:
  elasticsearch:
    container_name: elasticsearch
    image: 10.10.1.64:180/elasticsearch/elasticsearch:5.6.8
    restart: always
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 28g
    environment:
      - TZ=Asia/Shanghai
      - ES_JAVA_OPTS=-Xms2g -Xmx2g
    volumes:
      - ./conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./data:/usr/share/elasticsearch/data
      - ./logs:/usr/share/elasticsearch/logs
      - ./plugins:/usr/share/elasticsearch/plugins
      - ./snapshot:/usr/share/elasticsearch/snapshot
    network_mode: "host"

  elasticsearch-head:
    image: 10.10.1.64:180/elasticsearch/elasticsearch-head:5
    container_name: elasticsearch_header
    restart: always
    environment:
      - TZ=Asia/Shanghai
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 640m
    network_mode: "host"
```



4.3  运行

```bash
#docker-compose up -d

# docker ps
CONTAINER ID        IMAGE                                               COMMAND                  CREATED              STATUS              PORTS               NAMES
df87bede9484        10.10.1.64:180/elasticsearch/elasticsearch:5.6.8    "/docker-entrypoint.…"   About a minute ago   Up About a minute                       elasticsearch
d3f69a784ac9        10.10.1.64:180/elasticsearch/elasticsearch-head:5   "/bin/sh -c 'grunt s…"   About a minute ago   Up About a minute                       elasticsearch_header
```



## 五、验证

http://10.10.2.12:9100/

![360截图18151109155650.png](http://ww1.sinaimg.cn/large/007Xg1efgy1g9jqb81j85j31200bl3z4.jpg)