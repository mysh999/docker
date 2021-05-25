## 一、目的

prometheus监控启用热加载功能

prometheus使用docker部署





## 二、部署 

在docker-compose.yaml文件添加以下内容：

```bash
        command:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus"
            - "--web.console.libraries=/usr/share/prometheus/console_libraries"
            - "--web.console.templates=/usr/share/prometheus/consoles"
            - "--storage.tsdb.retention.time=3d"
            - "--web.enable-lifecycle"     #添加此参数
```





## 三、验证



执行以下命令热加载

```bash
#  curl -X POST http://127.0.0.1:9090/-/reload
```





