## 一、需求描述

emqx原本是容器化集群部署，但是没做日志映射输出

增加该功能



附emqx容器化文件：

```yaml
# cat docker-compose.yml 
version: '3'
services:
 emqx:
   image: emqx/emqx:v4.1.1
   container_name: emqx
   restart: always
   environment:
     - EMQX_LISTENER__TCP__EXTERNAL=1883
     - EMQX_LOADED_PLUGINS=emqx_management,emqx_auth_redis,emqx_recon,emqx_retainer,emqx_dashboard
     - EMQX_AUTH__MYSQL__SERVER=10.10.1.12:3306
     - EMQX_AUTH__MYSQL__USERNAME=root
     - EMQX_AUTH__MYSQL__PASSWORD=xxxxxxxxxxxx
     - EMQX_AUTH__MYSQL__DATABASE=mqtt
     - EMQX_AUTH__MYSQL__PASSWORD_HASH=sha256
     - EMQX_ALLOW_ANONYMOUS=false
     - EMQX_ACL_NOMAtCH=deny
     - EMQX_MQTT__MAX_PACKET_SIZE=10MB
     - EMQX_NAME=emqx
     - EMQX_HOST=lan12.emqx.io
     - EMQX_CLUSTER__DISCOVERY=static
     - EMQX_CLUSTER__STATIC__SEEDS=emqx@lan12.emqx.io,emqx@lan13.emqx.io,emqx@lan14.emqx.io
   extra_hosts:
     - "lan12.emqx.io:10.10.2.12"
     - "lan13.emqx.io:10.10.2.13"
     - "lan14.emqx.io:10.10.2.14"
   network_mode: host
```

三节点配置类似



## 二、解决办法

2.1、拷贝容器内的文件到容器外

```bash
docker cp emqx:/opt/emqx/etc ./etc
docker cp emqx:/opt/emqx/lib ./lib
docker cp emqx:/opt/emqx/data ./data
docker cp emqx:/opt/emqx/log ./log
```



2.2、修改属性

```bash
chown -R 1000:1000 data etc lib log
chmod -R 775 data etc lib log
```



2.3、增加卷映射

```yaml
# cat docker-compose.yml 
version: '3'
services:
 emqx:
   image: emqx/emqx:v4.1.1
   container_name: emqx
   restart: always
   environment:
     - EMQX_LISTENER__TCP__EXTERNAL=1883
     - EMQX_LOADED_PLUGINS=emqx_management,emqx_auth_redis,emqx_recon,emqx_retainer,emqx_dashboard
     - EMQX_AUTH__MYSQL__SERVER=10.10.1.12:3306
     - EMQX_AUTH__MYSQL__USERNAME=root
     - EMQX_AUTH__MYSQL__PASSWORD=xxxxxxxxxxxxxxx
     - EMQX_AUTH__MYSQL__DATABASE=mqtt
     - EMQX_AUTH__MYSQL__PASSWORD_HASH=sha256
     - EMQX_ALLOW_ANONYMOUS=false
     - EMQX_ACL_NOMAtCH=deny
     - EMQX_MQTT__MAX_PACKET_SIZE=10MB
     - EMQX_NAME=emqx
     - EMQX_HOST=lan12.emqx.io
     - EMQX_CLUSTER__DISCOVERY=static
     - EMQX_CLUSTER__STATIC__SEEDS=emqx@lan12.emqx.io,emqx@lan13.emqx.io,emqx@lan14.emqx.io
   extra_hosts:
     - "lan12.emqx.io:10.10.2.12"
     - "lan13.emqx.io:10.10.2.13"
     - "lan14.emqx.io:10.10.2.14"
   volumes:
     - ./lib:/opt/emqx/lib
     - ./etc:/opt/emqx/etc
     - ./data:/opt/emqx/data
     - ./log:/opt/emqx/log
   network_mode: host
```



2.4、重启容器

```bash
# docker-compose down;docker-compose up -d --build;docker-compose logs -f
```

