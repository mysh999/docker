docker是直接走iptables转发，不受自带防火墙的控制
所以关闭iptables或者开放，仍存在不通的现象

解决办法：
```bash
# vi /usr/lib/systemd/system/docker.service
将
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

改成
ExecStart=/usr/bin/dockerd --iptables=false -H fd:// --containerd=/run/containerd/containerd.sock
```
再重启服务
```bash
systemctl daemon-reload
systemctl restart docker
```

