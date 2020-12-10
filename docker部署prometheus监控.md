docker部署prometheus监控





[TOC]



## 一、环境说明

环境部署了docker和docker-compose插件

在10.10.2.200 环境中部署prometheus服务端



- 使用node-exporters插件监控各系统

- 使用balckbox-exporters插件监控各应用url
- 使用gpu_prometheus_exporter插件监控GPU工作情况

- 使用grafana做展示



在测试、开发、线上环境部署客户端监控



## 二、服务端部署



### 2.1、拉取镜像

```bash
# docker pull prom/prometheus
# docker pull quay.io/prometheus/node-exporter
# docker pull grafana/grafana
# docker pull prom/blackbox-exporter
# docker pull prom/alertmanager
# docker pull google/cadvisor
# docker pull prom/mysqld-exporter



# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
grafana/grafana                    latest              b71c098643ff        3 months ago        180MB
wordpress                          latest              f1da35a7ddca        3 months ago        546MB
prom/prometheus                    latest              b205ccdd28d3        4 months ago        145MB
mysql                              5.7                 718a6da099d8        4 months ago        448MB
prom/blackbox-exporter             latest              fe4b2bc20600        5 months ago        21.7MB
prom/alertmanager                  latest              c876f5897d7b        5 months ago        55.5MB
quay.io/prometheus/node-exporter   latest              0e0218889c33        5 months ago        26.4MB
prom/blackbox-exporter             v0.16.0             5e5949f1bda7        13 months ago       19.7MB
prom/mysqld-exporter               latest              432cdd0a0e7c        16 months ago       17.5MB
google/cadvisor                    latest              eb1210707573        2 years ago         69.6MB
gliderlabs/registrator             latest              3b59190c6c80        4 years ago         23.8MB
```



###  2.2、配置服务端

```bash
# mkdir -p /data/ubt/prom/prometheus

# cat /data/ubt/prom/prometheus/config/prometheus.yml 
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['10.10.2.200:9093']
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  - "*rules.yml"
  - "*alert.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
      - targets: ['10.10.2.200:9090']

  - job_name: '200_docker'
    static_configs:
      - targets: ['10.10.2.200:8080']

  - job_name: 'prom_node'
    scrape_interval: 8s
    static_configs:
      - targets: ['10.10.2.200:9100']


  - job_name: 'bigdata_node12'
    scrape_interval: 8s
    static_configs:
      - targets: ['10.10.2.12:9100']

 

  - job_name: '测试环境_url_monitor'
    scrape_interval: 30s
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://xx.xx.com/map/
        - https://xx.xx.com/v1/gdpr/
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.10.2.200:9115

```



### 2.3、对应的docker-compose文件

```bash
# cat /data/ubt/prom/prometheus/docker-compose.yml 
version: '3.7'

networks:
    monitor:
        driver: bridge

services:
    prometheus:
        image: prom/prometheus
        container_name: prometheus
        hostname: prometheus
        restart: always
        volumes:
            - ./config:/etc/prometheus
            - ./prometheus_data:/prometheus
        ports:
            - "9090:9090"
        networks:
            - monitor

    alertmanager:
        image: prom/alertmanager
        container_name: alertmanager
        hostname: alertmanager
        restart: always
        volumes:
            - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
        ports:
            - "9093:9093"
        networks:
            - monitor

    grafana:
        image: grafana/grafana
        container_name: grafana
        hostname: grafana
        restart: always
        ports:
            - "3000:3000"
        volumes:
            - ./grafata_data:/var/lib/grafana
        networks:
            - monitor

    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: always
        ports:
            - "9100:9100"
        networks:
            - monitor

    cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        hostname: cadvisor
        restart: always
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:rw
            - /sys:/sys:ro
            - ./docker_containers/:/var/lib/docker:ro
        ports:
            - "8080:8080"
        networks:
            - monitor

    mysql-exporter:
        image: prom/mysqld-exporter
        volumes:
           - /etc/timezone:/etc/timezone
           - /etc/localtime:/etc/localtime:ro
        container_name: mysql-exporter
        hostname: mysql-exporter
        restart: always
        ports:
            - "9104:9104"
        networks:
            - monitor 
 
        environment:
            - DATA_SOURCE_NAME=root:password@(10.10.2.200:3306)/mysql

    black-exporter:
        image: prom/blackbox-exporter:v0.16.0
        container_name: blackbox 
        volumes:
           - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
           - ./blackbox.yml:/config/blackbox.yml
        command:
           - '--config.file=/config/blackbox.yml'
        ports:
           - "9115:9115"
        networks:
           - monitor
```





### 2.4、对应的告警配置

```bash
# cat /data/ubt/prom/prometheus/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.xxxxxxxx.com:465' 
  smtp_from: 'cloud_services@xxxxxxxx.com'
  smtp_auth_username: 'cloud_services@xxxxxxxx.com' 
  smtp_auth_password: 'XXXXXXXXXX'  
  smtp_require_tls: false
  wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'  
  wechat_api_corp_id: 'XXXXXXXXXXXXXXXXX' 
templates:
- '/var/lib/alertmanager/*.tmpl'  
route:
  receiver: "email" 
  group_by: ['env','instance','alertname','type','group','job']
  group_wait:      30s
  group_interval:  3m
  repeat_interval: 12h
  routes:
  - receiver: "wechat"
    match:
      severity: test
receivers:
- name: 'email' 
  email_configs: 
  - to: 'aaa@xxxxxxxx.com' 
    send_resolved: true   
- name: 'wechat' 
  wechat_configs:
  - send_resolved: true  
    to_user: '@all'  
    message: '{{ template "wechat.default.message" . }}'  
    agent_id: '1000002'  
    api_secret: 'XXXXXXXXXXXXXXXXXXX' 
```



### 2.5、对应告警内容

```bash
# cat /data/ubt/prom/prometheus/config/alert-rules.yml 
groups:
- name: node
  rules:
  - alert: 系统CPU使用率大于95%
    expr: (1-rate(node_cpu_seconds_total{mode="idle"}[1m]))*100 > 95
    for: 2m #持续2分都是95%以上
    labels:
      severity: page
    注释:
      描述: "当前可用值: {{ $value }}"
  - alert: 系统内存可使用率低于10%
    expr: node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes*100 < 10 
    for: 2m
    labels:
      severity: page
    注释:
      description: "current value: {{ $value }}"
  - alert: 系统CPU负载大于80
    expr: node_load1 > 80
    for: 2m
    labels:
      severity: page
    注释:
      描述: "当前可用值: {{ $value }}"
  - alert: 磁盘空间使用率低于10%
    #expr: node_filesystem_avail_bytes{device!='nsfs'}/node_filesystem_size_bytes{device!='nsfs'}*100 < 10
    expr: node_filesystem_avail_bytes{fstype=~"ext4|xfs"}/node_filesystem_size_bytes{fstype=~"ext4|xfs"}*100 < 10
    for: 2m
    labels:
      severity: page
    注释:
      描述: "当前可用值: {{ $value }}"
```



```bash
# cat /data/ubt/prom/prometheus/config/node_down.yml 
groups:
- name: node_down
  rules:
  - alert: 宕机警告 
    expr: up == 0
    for: 1m
    labels:
      user: test
    annotations:
      summary: "实例 {{ $labels.instance }} 已宕机"
      description: "{{ $labels.instance }} of job {{ $labels.job }} 宕机已经超过一分钟，请检查确认！！！"
```





### 2.6、启动

```bash
# docker-compose up -d
```







## 三、客户端配置



运行node-exporter和cadvisor

```bash
# cat /data/ubt/promm/docker-compose.yml 
version: '3.3'

networks:
    monitor:
        driver: bridge

services:
    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: always
        ports:
            - "9200:9100"
        networks:
            - monitor

    cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        hostname: cadvisor
        restart: always
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:ro
            - /sys:/sys:ro
            - /dev/disk/:/dev/disk:ro
            - /var/lib/docker/:/var/lib/docker:ro
        ports:
            - "18080:8080"
        networks:
            - monitor

```



运行gpu插件

```bash
#运行gpu插件
[root@cloud-nlp ~]# nvidia-docker run -d -p 9445:9445 -ti mindprince/nvidia_gpu_prometheus_exporter:0.1
```





查看prom

```bash
http://10.10.2.200:9090/targets
```

![企业微信截图_20201210141517.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gliqgxwqk1j31du0q10uh.jpg)





## 四、配置grafana



配置grafana

```bash
http://10.10.2.200:3000
```

选择data source

![20190902_1135_54s.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1g6l0prtaqnj30lz0jejuk.jpg)



添加插件

```bash
http://10.10.2.200:3000/dashboard/import
```

![20190902_1136_38s.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1g6l0q9h2f5j30m10lr44f.jpg)





![20190902_1137_47s.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1g6l0ri081fj30lx0b0jt8.jpg)



配置饼图

默认没有饼图插件，需要加载

```bash
[root@cloud-72 config]# docker exec -i grafana sh -c 'grafana-cli plugins install grafana-piechart-panel'
installing grafana-piechart-panel @ 1.3.8
from: https://grafana.com/api/plugins/grafana-piechart-panel/versions/1.3.8/download
into: /var/lib/grafana/plugins

✔ Installed grafana-piechart-panel successfully 

Restart grafana after installing plugins . <service grafana-server restart>


[root@cloud-72 config]# docker restart grafana
```



效果：

![20190902_1138_15s.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1g6l0ry9abxj30m10abq7n.jpg)







![企业微信截图_20201210141751.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gliqjgely8j30yf0ipab0.jpg)