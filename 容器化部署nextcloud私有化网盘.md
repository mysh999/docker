## 一、配置docker-compose

```bash
# cat docker-compose.yml 
version: '2'
services:
   db:
     image: mariadb
     command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
     restart: always
     volumes:
       - ./db:/var/lib/mysql
     environment:
       - MYSQL_ROOT_PASSWORD=a123456
       - MYSQL_PASSWORD=a123456
       - MYSQL_DATABASE=nextclouddb
       - MYSQL_USER=nextclouduser
   app:
     image: nextcloud
     ports:
       - 1234:80
     links:
       - db
     volumes:
       - ./nextcloud:/var/www/html
       - ./data:/var/www/html/data
     restart: always
```

执行

```bash
#docker-compose up -d
```



访问：

http://10.10.1.190:1234/

使用用户名：nextclouduser

 密码: a123456

登陆



## 二、配置nginx

安装nginx

```bash
#yum -y install nginx
```



nginx配置

```bash
# egrep -v "#" nginx.conf 

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 102400;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        include /etc/nginx/default.d/*.conf;


        location / {
            proxy_next_upstream error timeout http_503 http_500 http_502 http_504;
            proxy_pass http://127.0.0.1:1234;
            proxy_redirect off;
            proxy_store off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_ignore_client_abort on;
            proxy_intercept_errors on;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }


}
```



访问：

```bash
http://10.10.1.190
```

