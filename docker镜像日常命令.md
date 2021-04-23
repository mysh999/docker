查看镜像

```bash
# docker images
```





使用tag命令为镜像添加标签

```bash
# docker tag centos:7.3.1611 centos:new
```



查看镜像详细信息

```bash
# docker inspect tomcat:8.5.23
```



查看镜像各层具体内容历史

```bash
# docker history tomcat:8.5.23
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
cef8b6ca1cdc        2 years ago         /bin/sh -c #(nop)  ENTRYPOINT ["/bin/sh" "...   0 B                 
d1b5221668a7        2 years ago         /bin/sh -c #(nop)  EXPOSE 8080/tcp              0 B                 
f7f00b372848        2 years ago         /bin/sh -c #(nop) WORKDIR /usr/local/apach...   0 B                 
c716c966aaff        2 years ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/jdk...   0 B                 
591d8449116c        2 years ago         /bin/sh -c #(nop)  ENV CATALINA_HOME=/usr/...   0 B                 
03aa220535db        2 years ago         /bin/sh -c #(nop)  ENV CLASSPATH=.:/usr/lo...   0 B                 
315bbece0e86        2 years ago         /bin/sh -c #(nop)  ENV JRE_HOME=/usr/local...   0 B                 
52474bb60b3d        2 years ago         /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/loca...   0 B                 
b6c6355f8c2a        2 years ago         /bin/sh -c #(nop)  ENV LC_ALL=en_US.UTF-8       0 B                 
42f70b898e10        2 years ago         /bin/sh -c #(nop)  ENV LANG=en_US.UTF-8         0 B                 
f2b1719a4b01        2 years ago         /bin/sh -c #(nop) ADD file:a96dea0ad1e52c1...   384 MB              
d0346c808cb1        2 years ago         /bin/sh -c #(nop) ADD file:3d26c6e3129489f...   13.4 MB             
488089961709        2 years ago         /bin/sh -c /bin/cp /usr/share/zoneinfo/Asi...   402 B               
ccacf52e64a3        2 years ago         /bin/sh -c #(nop)  MAINTAINER "kaiqing.hua...   0 B                 
c19fd3d6ea99        2 years ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B                 
c5c92002e5fb        2 years ago         /bin/sh -c #(nop)  LABEL name=CentOS Base ...   0 B                 
3916edf2e446        2 years ago         /bin/sh -c #(nop) ADD file:f7abac43ddf5433...   192 MB     
```

查看更具体的信息

```bash
# docker history --no-trunc  tomcat:8.5.23
```

