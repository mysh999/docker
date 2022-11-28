```bash
$ cat docker-compose.yml 
version: '3'
services:
    mysql:
        network_mode: "bridge"
        environment:
            MYSQL_ROOT_PASSWORD: "123456"
            MYSQL_USER: 'test'
            MYSQL_PASS: '123456'
        image: "mysql:5.7"
        restart: always
        volumes:
            - "./db:/var/lib/mysql"
            - "./conf/my.cnf:/etc/my.cnf"
            - "./init:/docker-entrypoint-initdb.d/"
        ports:
            - "13306:3306"

$ cat ./conf/my.cnf 
[mysqld]
server-id=1000
user=mysql
default-storage-engine=INNODB
character-set-server=utf8
log-bin=/var/lib/mysql/mysql-bin
expire_logs_days=3
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8


$ cat ./init/init.sql 
create database test;
use test;
create table user
(
id int auto_increment primary key,
username varchar(64) unique not null,
email varchar(120) unique not null,
password_hash varchar(128) not null,
avatar varchar(128) not null
);
insert into user values(1, "zhangsan","test12345@qq.com","passwd","avaterpath");
insert into user values(2, "lisi","12345test@qq.com","passwd","avaterpath");
```

