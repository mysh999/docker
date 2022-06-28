## 一、问题描述

 docker启动的时候报：

```bash
ems-micro-service                    |  at java.lang.reflect.Method.invoke(Method.java:498)
ems-micro-service                    |  at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
ems-micro-service                    |  at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
ems-micro-service                    |  at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
ems-micro-service                    |  at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
ems-micro-service                    | Caused by: com.alibaba.dubbo.remoting.RemotingException: Failed to bind NettyServer on /172.31.0.111:20880, cause: Failed to bind to: /0.0.0.0:20880
ems-micro-service                    |  at com.alibaba.dubbo.remoting.transport.AbstractServer.<init>(AbstractServer.java:68)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.transport.netty.NettyServer.<init>(NettyServer.java:61)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.transport.netty.NettyTransporter.bind(NettyTransporter.java:32)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.Transporter$Adaptive.bind(Transporter$Adaptive.java)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.Transporters.bind(Transporters.java:56)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchanger.bind(HeaderExchanger.java:44)
ems-micro-service                    |  at com.alibaba.dubbo.remoting.exchange.Exchangers.bind(Exchangers.java:70)
ems-micro-service                    |  at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.createServer(DubboProtocol.java:284)
```



##   二、解决办法

在启动参数里添加-Ddubbo.protocol.port参数

```bash
# vi docker-compose.yml
... ...
    environment:
      - TZ=Asia/Shanghai
      - PROFILES=inland
      - RUNS_OPTS=-Xms512m -Xmx512m -Dserver.port=8024 -Ddubbo.protocol.port=28191
      - HEALTH_PORT=8024
```

