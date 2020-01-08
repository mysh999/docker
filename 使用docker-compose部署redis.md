1、创建目录

```bash
#mkdir -p config data logs
```



2、准备redis.conf配置文件

```bash
# cat ./config/redis.conf 
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
logfile /redis/log/redis.log
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



3、准备docker-compose文件

```bash


# cat docker-compose.yml 
redis:
  image: redis:4.0.11
  container_name: my_redis
  environment:
    TZ: Asia/Shanghai
  command: redis-server /usr/local/etc/redis/redis.conf
  ports:
    - "6379:6379"
  volumes:
    - ./data:/data
    - ./config/redis.conf:/usr/local/etc/redis/redis.conf
    - ./logs:/redis/log
  restart: always
```



4、执行部署

```bash
#docker-compose up -d
```



5、查看

```bash
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS                    NAMES
6a03f543f239        redis:4.0.11        "docker-entrypoint.s…"   21 minutes ago      Up 21 minutes                   0.0.0.0:6379->6379/tcp   my_redis
```

