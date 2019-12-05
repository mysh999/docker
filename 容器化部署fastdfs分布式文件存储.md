## 一、目的

使用容器化技术，部署fastdfs分布式文件存储环境

这里以部署单机环境为例。



## 二、文件结构

```bash
# tree .
.
├── configure
│   ├── client.conf
│   ├── config.tar.gz
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── fastdfs-5.05.tar.gz
│   ├── fastdfs-nginx-module-master.tar.gz
│   ├── libfastcommon-1.0.7.tar.gz
│   ├── mod_fastdfs.conf
│   ├── nginx-1.12.1.tar.gz
│   ├── nginx.conf
│   ├── repo.tar.gz
│   ├── storage.conf
│   └── tracker.conf
└── docker-compose.yml
```



## 三、配置文件

```bash
# cat docker-compose.yml 
version: '3.1'
services:
  fastdfs:
    build: configure
    image: fastdfs:v1
    restart: always
    container_name: fastdfs
    volumes:
      - ./storage:/fastdfs/storage
      - ./logs:/usr/local/nginx/logs
      - ./data:/ljzsg/fastdfs/file/data
    ports:
      - 18080:80
    networks:
      - default

networks:
  default:
    external:
      name: dfs
      
      
     
```



```bash
# cat ./configure/Dockerfile 
FROM centos:centos7.5.1804
MAINTAINER miyong.shu@ubtrobot.com


# 安装依赖
RUN yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel perl initscripts

# 复制工具包
ADD fastdfs-5.05.tar.gz /usr/local/src
ADD fastdfs-nginx-module-master.tar.gz /usr/local/src
ADD libfastcommon-1.0.7.tar.gz /usr/local/src
ADD nginx-1.12.1.tar.gz /usr/local/src

# 安装 libfastcommon
WORKDIR /usr/local/src/libfastcommon-1.0.7
RUN ./make.sh && ./make.sh install
RUN ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
RUN ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so

# 安装 FastDFS
WORKDIR /usr/local/src/fastdfs-5.05
RUN ./make.sh && ./make.sh install
RUN ln -s /usr/bin/fdfs_trackerd   /usr/local/bin
RUN ln -s /usr/bin/fdfs_storaged   /usr/local/bin
RUN ln -s /usr/bin/stop.sh         /usr/local/bin
RUN ln -s /usr/bin/restart.sh      /usr/local/bin

# 配置 FastDFS 跟踪器
ADD tracker.conf /etc/fdfs
RUN mkdir -p /ljzsg/fastdfs/tracker

# 配置 FastDFS 存储
ADD storage.conf /etc/fdfs
RUN mkdir -p /ljzsg/fastdfs/storage
RUN mkdir -p /ljzsg/fastdfs/file

# 配置 FastDFS 客户端
ADD client.conf /etc/fdfs
RUN mkdir -p /ljzsg/fastdfs/client

# 配置 fastdfs-nginx-module
ADD mod_fastdfs.conf /usr/local/src/fastdfs-nginx-module-master/src

# FastDFS 与 Nginx 集成
WORKDIR /usr/local/src/nginx-1.12.1
RUN ./configure --add-module=/usr/local/src/fastdfs-nginx-module-master/src
RUN make && make install
ADD mod_fastdfs.conf /etc/fdfs/mod_fastdfs.conf

WORKDIR /usr/local/src/fastdfs-5.05/conf
RUN cp anti-steal.jpg http.conf mime.types /etc/fdfs/

# 配置 Nginx
ADD nginx.conf /usr/local/nginx/conf

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

WORKDIR /
EXPOSE 80
CMD ["/bin/bash"]
```



```bash
# cat ./configure/entrypoint.sh 
#!/bin/sh
/etc/init.d/fdfs_trackerd start
/etc/init.d/fdfs_storaged start
/usr/local/nginx/sbin/nginx -g 'daemon off;'
```



```bash
# cat ./configure/client.conf |egrep -v "^#"|egrep -v "^$" 
connect_timeout=30
network_timeout=60
base_path=/ljzsg/fastdfs/client
tracker_server=fastdfs:22122
log_level=info
use_connection_pool = false
connection_pool_max_idle_time = 3600
load_fdfs_parameters_from_tracker=false
use_storage_id = false
storage_ids_filename = storage_ids.conf
http.tracker_server_port=80
```



```bash
# cat ./configure/tracker.conf |egrep -v "^#"|egrep -v "^$"      
disabled=false
bind_addr=
port=22122
connect_timeout=30
network_timeout=60
base_path=/ljzsg/fastdfs/tracker
max_connections=256
accept_threads=1
work_threads=4
store_lookup=2
store_group=group2
store_server=0
store_path=0
download_server=0
reserved_storage_space = 10%
log_level=info
run_by_group=
run_by_user=
allow_hosts=*
sync_log_buff_interval = 10
check_active_interval = 120
thread_stack_size = 64KB
storage_ip_changed_auto_adjust = true
storage_sync_file_max_delay = 86400
storage_sync_file_max_time = 300
use_trunk_file = false 
slot_min_size = 256
slot_max_size = 16MB
trunk_file_size = 64MB
trunk_create_file_advance = false
trunk_create_file_time_base = 02:00
trunk_create_file_interval = 86400
trunk_create_file_space_threshold = 20G
trunk_init_check_occupying = false
trunk_init_reload_from_binlog = false
trunk_compress_binlog_min_interval = 0
use_storage_id = false
storage_ids_filename = storage_ids.conf
id_type_in_filename = ip
store_slave_file_use_link = false
rotate_error_log = false
error_log_rotate_time=00:00
rotate_error_log_size = 0
log_file_keep_days = 0
use_connection_pool = false
connection_pool_max_idle_time = 3600
http.server_port=80
http.check_alive_interval=30
http.check_alive_type=tcp
http.check_alive_uri=/status.html
```



```bash
# cat ./configure/storage.conf |egrep -v "^#"|egrep -v "^$"            
disabled=false
group_name=group1
bind_addr=
client_bind=true
port=23000
connect_timeout=30
network_timeout=60
heart_beat_interval=30
stat_report_interval=60
base_path=/ljzsg/fastdfs/storage
max_connections=256
buff_size = 256KB
accept_threads=1
work_threads=4
disk_rw_separated = true
disk_reader_threads = 1
disk_writer_threads = 1
sync_wait_msec=50
sync_interval=0
sync_start_time=00:00
sync_end_time=23:59
write_mark_file_freq=500
store_path_count=1
store_path0=/ljzsg/fastdfs/file
subdir_count_per_path=256
tracker_server=fastdfs:22122
log_level=info
run_by_group=
run_by_user=
allow_hosts=*
file_distribute_path_mode=0
file_distribute_rotate_count=100
fsync_after_written_bytes=0
sync_log_buff_interval=10
sync_binlog_buff_interval=10
sync_stat_file_interval=300
thread_stack_size=512KB
upload_priority=10
if_alias_prefix=
check_file_duplicate=0
file_signature_method=hash
key_namespace=FastDFS
keep_alive=0
use_access_log = false
rotate_access_log = false
access_log_rotate_time=00:00
rotate_error_log = false
error_log_rotate_time=00:00
rotate_access_log_size = 0
rotate_error_log_size = 0
log_file_keep_days = 0
file_sync_skip_invalid_record=false
use_connection_pool = false
connection_pool_max_idle_time = 3600
http.domain_name=
http.server_port=80
```



```bash
# cat configure/nginx.conf |egrep -v "^#"|egrep -v "^$"
worker_processes  auto;
events {
    worker_connections  10240;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;                   #sendfile提高 Nginx 静态资源托管效率。直接在内核空间完成文件发送，不需要先 read 再 write，没有上下文切换开销
    tcp_nopush      on;                   #只有在启用了 sendfile 之后才生效。启用它之后，数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率
    tcp_nodelay     on;
    keepalive_timeout  65;
    client_max_body_size    256m;         #修改上传文件大小限制为256M
    server_tokens     off;                #隐藏版本号
    gzip on;                              #启用压缩
    gzip_min_length   1k;
    gzip_buffers      4 16k;
    gzip_http_version 1.0;
    gzip_comp_level   2;
    gzip_types        text/plain application/x-javascript text/css application/xml text/javascript application/javascript;
    gzip_vary         on;
    server {
        listen       80;
        server_name  localhost;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location /group([0-9])/M00 {
            alias /ljzsg/fastdfs/file/data;
        }
        location ~/group([0-9])/M00 {
                ngx_fastdfs_module;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```



```bash
# cat ./configure/mod_fastdfs.conf |egrep -v "^#"|egrep -v "^$"          
connect_timeout=10
network_timeout=30
base_path=/tmp
load_fdfs_parameters_from_tracker=true
storage_sync_file_max_delay = 86400
use_storage_id = false
storage_ids_filename = storage_ids.conf
tracker_server=fastdfs:22122
storage_server_port=23000
group_name=group1
url_have_group_name = true
store_path_count=1
store_path0=/ljzsg/fastdfs/file
log_level=info
log_filename=
response_mode=proxy
if_alias_prefix=
flv_support = true
flv_extension = flv
group_count = 0
```



## 四、运行

```bash
#docker-compose up -d
# docker ps
CONTAINER ID        IMAGE                                                                COMMAND                  CREATED             STATUS                      PORTS                    NAMES
0398eca85dcd        fastdfs:v1                                                           "/usr/local/bin/en..."   2 minutes ago       Up 2 minutes                0.0.0.0:18080->80/tcp    fastdfs

```



## 五、上传文件测试

```bash
#上传一张图片到宿主机的./storage目录下

# docker exec -it 0398eca85dcd sh
sh-4.2# cd /fastdfs/storage
sh-4.2# ls
a.jpg
sh-4.2# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf a.jpg     
group1/M00/00/00/wKgQAl3cj3OAcDSUAAX_vvMAn1o434.jpg    #上传成功

#打开
浏览器打开
http://10.10.1.61:18080/group1/M00/00/00/wKgQAl3cj3OAcDSUAAX_vvMAn1o434.jpg
#可以加载图片
```

![360截图17290428526489.png](http://ww1.sinaimg.cn/large/007Xg1efgy1g9b8oydeyej31cg0rr4qp.jpg)



## 六、后记

相关介质配置文件，打包在fastdfs.tar.gz文件中

https://github.com/mysh1984/docker/raw/master/src/fastdfs.tar.gz



参考文档：

[http://nginx.org/en/docs/http/ngx_http_core_module.html]

