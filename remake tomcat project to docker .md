## 一、object

have a standard tomcat8's project，will update it to  Tomcat 8.5.59 and use docker deployment



tomcat8's project directory list:

```bash
 apache-tomcat-8.0.30]# ll
total 96
drwxrwxr-x  2 xxx   xxx  4096 Nov 23 16:55 bin
drwxrwxr-x  3 xxx   xxx   198 Nov 23 17:07 conf
drwxrwxr-x  2 xxx   xxx   115 May 10  2019 jimuLogs
drwxrwxr-x  2 xxx   xxx  4096 May 10  2019 lib
-rwxrwxr-x  1 xxx   xxx 57011 Dec  2  2015 LICENSE
drwxrwxr-x  2 xxx   xxx   177 Nov 23 17:07 logs
-rwxrwxr-x  1 xxx   xxx  1444 Dec  2  2015 NOTICE
-rwxrwxr-x  1 xxx   xxx  6741 Dec  2  2015 RELEASE-NOTES
-rwxrwxr-x  1 xxx   xxx 16195 Dec  2  2015 RUNNING.txt
drwxrwxr-x  2 xxx   xxx    30 Sep 17  2019 temp
drwxrwxr-x 11 xxx   xxx   206 Jul 17 13:55 webapps
drwxrwxr-x  3 xxx   xxx    22 May 10  2019 work
```





## 二、disable AJP

because higher version's AJP is clash for project,so disable it

```bash
# vi ./conf/server.xml 
# comment it 
    <!--
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    -->
```



## 三、configure with  docker

```bash
# cat docker-compose.yaml 
version: '2'
services:
  xxx-service:
    container_name: xxx-service
    image: xxxxx/tomcat:8.5.59
    restart: always
    mem_limit: 3g
    environment: 
      - TZ=Asia/Shanghai
      - JAVA_OPTS=-Xmx1g -Xms1g
    network_mode: host
    volumes:
      - ./conf:/usr/local/tomcat/conf
      - ./webapps:/usr/local/tomcat/webapps
      - ./logs:/usr/local/tomcat/logs

```



## 四、run 

```bash
#docker-compose up -d
```

