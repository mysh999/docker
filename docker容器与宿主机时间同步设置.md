将docker容器/etc/localtime 和/etc/timezone 与宿主机文件共享就能实现时间同步
只要修改docker-compose.yml文件

```yaml
volumes:
   - /etc/timezone:/etc/timezone
   - /etc/localtime:/etc/localtime
```
