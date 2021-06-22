## 一、nginx安装监控模块

这里nginx采用二进制方式安装

查看nginx信息

```bash
# nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```

可知nginx默认安装路径是/usr/share/nginx



下载官方版的nginx-module-vts模块

```bash
$ git clone git://github.com/vozlt/nginx-module-vts.git
```



下载同版本可编译的nginx源码

```bash
# cd /opt
# wget http://nginx.org/download/nginx-1.16.1.tar.gz  
# tar -zxvf nginx-1.16.1.tar.gz 
# cd nginx-1.16.1
```





备份原nginx文件

```bash
# mv /usr/sbin/nginx /usr/sbin/nginx.bak
# cp -rf /etc/nginx{,.bak}
```



安装编译必备软件

```bash
# yum -y install gcc pcre-devel openssl-devel libxslt-devel libxml2 gd-devel geoip-devel perl-devel perl-ExtUtils-Embed
```





重新编译nginx

  查模块是否支持，比如这次添加 limit 限流模块 和 stream 模块：
./configure –-help | grep limit
这里写图片描述
ps：-without-http_limit_conn_module disable 表示已有该模块，编译时，不需要添加

./configure –-help | grep stream
这里写图片描述
ps：–with-stream enable 表示不支持，编译时要自己添加该模块  

```bash
# ./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_image_filter_module=dynamic --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-module=/root/nginx-module-vts/
```

configure参数使用nginx -V获取的配置

最后加上

-add-module=/root/nginx-module-vts/



编译通过，继续

```bash
# make
# make install
```



验证

以上完成后，会在objs目录下生成一个nginx文件

```bash
[root@localhost nginx-1.16.1]# /opt/nginx-1.16.1/objs/nginx -t
nginx: [emerg] module "/usr/lib64/nginx/modules/ngx_http_image_filter_module.so" is not binary compatible in /usr/share/nginx/modules/mod-http-image-filter.conf:1
nginx: configuration file /etc/nginx/nginx.conf test failed
[root@localhost nginx-1.16.1]# /opt/nginx-1.16.1/objs/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --add-module=/root/nginx-module-vts/
```





替换nginx文件并重启

```bash
# cp /opt/nginx-1.16.1/objs/nginx /usr/sbin/
```







nginx添加以下配置内容

加在http和server模块中

```bash
# cat /etc/nginx/nginx.conf
... ...

http {
    vhost_traffic_status_zone;    #新增
    vhost_traffic_status_filter_by_host on;  #新增
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

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        vhost_traffic_status off;       #新增                 
        location /status {               #新增
                vhost_traffic_status_display;   #新增
                vhost_traffic_status_display_format html;   #新增
        }
        ... ...
```





验证

在浏览器中输入

```html
http://10.10.2.200/status
```

![企业微信截图_20210622194124.png](http://ww1.sinaimg.cn/large/007Xg1efgy1grra42pwkgj60vw0bfmxp02.jpg)





## 二、部署nginx-vts-exporter

exporter会收集nginx性能指标的JSON格式数据，并汇总后暴露监控接口给Prometheus



执行以下命令：

```bash
# wget -c https://github.com/hnlq715/nginx-vts-exporter/releases/download/v0.9.1/nginx-vts-exporter-0.9.1.linux-amd64.tar.gz
# tar -zxvf nginx-vts-exporter-0.9.1.linux-amd64.tar.gz 

# 启动
# nohup nginx-vts-exporter-0.9.1.linux-amd64/nginx-vts-exporter -nginx.scrape_timeout 10 -nginx.scrape_uri http://10.10.2.200/status/format/json &

```

nginx_vts_exporter的默认端口号：9913，对外暴露监控接口http://xxx:9913/metrics

打开http://10.10.2.200:9913/metrics

显示收集的数据

![企业微信截图_20210622201703.png](http://ww1.sinaimg.cn/large/007Xg1efgy1grrb4ystwuj60yn0r10u302.jpg)

## 三、配置prometheus

在prometheus配置文件中追加以下内容

```bash
# vi prometheus.yml 
# 追加以下内容
  - job_name: '10.10.2.200_nginx-vts'
    static_configs:
      - targets: ['10.10.2.200:9913']
```

热加载 

```html
# curl -X POST http://10.10.2.200:9090/-/reload
```



打开以下链接显示出nginx对象

http://10.10.2.200:9090/targets

![企业微信截图_20210622201858.png](http://ww1.sinaimg.cn/large/007Xg1efgy1grrb6yi2utj616v0baaam02.jpg)





## 四、grafana配置

使用2949模板

![企业微信截图_20210622201936.png](http://ww1.sinaimg.cn/large/007Xg1efgy1grrb7l5ocbj61ev0qzq5m02.jpg)