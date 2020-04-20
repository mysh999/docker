docker与防火墙关系

说明：
docker在防火墙启用的情况下，需要打开ip_forward转发功能；如果防火墙关闭，使用内核转发
docker采用默认桥接网络方式，对端口做映射 

- 测试一：
  关闭防火墙（iptable或者firewalld）下

通信情况：
外部可以和docker建立通信





- 测试二：
  启用防火墙（iptable或者firewalld）下

```bash
# systemctl start firewalld 
```



通信情况：
外部不可以和docker建立通信





- 测试三：
  启用防火墙（firewalld）下

```bash
# systemctl start firewalld.service
```



打开ip转发功能, 下面两种方法都是临时打开ip转发功能!
```bash
# echo 1 > /proc/sys/net/ipv4/ip_forward

# sysctl -w net.ipv4.ip_forward=1
```



永久生效的ip转发
```bash
# vim /etc/sysctl.conf
net.ipv4.ip_forward = 1

# sysctl -p /etc/sysctl.conf      // 立即生效
```



通信情况：
外部不可以和docker建立通信





- 测试四：
  在测试三的基础上配置firewalld防火墙

  ```bash
  #firewall-cmd --add-rich-rule="rule family="ipv4" source address="0.0.0.0/0" accept" --permanent
  #iptables-save
  ```

  

  通信情况：
  外部可以和docker建立通信
  备注：如果操作系统重启后，启动docker-compose报ERROR: Failed to Setup IP tables: Unable to enable SKIP DNAT rule ... iptables: No chain/target/match by that name
  解决办法：
  重启docker服务
  #systemctl restart docker



- 测试五：
  启用防火墙（iptable）下和打开ip转发功能

```bash
# systemctl start iptables.service
```



打开ip转发功能, 下面两种方法都是临时打开ip转发功能!
```bash
# echo 1 > /proc/sys/net/ipv4/ip_forward

# sysctl -w net.ipv4.ip_forward=1
```



永久生效的ip转发
```bash
# vim /etc/sysctl.conf

net.ipv4.ip_forward = 1

# sysctl -p /etc/sysctl.conf      // 立即生效
```

通信情况：
外部不可以和docker建立通信



- 测试六：
  在测试五的基础上配置iptables防火墙

```bash
# iptables -F

# iptables -A FORWARD -i docker0 -j ACCEPT

# service iptables save      #保存配置
```



通信情况：
外部可以和docker建立通信