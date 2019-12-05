## 一、原理

mysql配置互为主从（双主模式）



## 二、前提

1、2台数据库服务器，ID不一样

2、启用二进制日志

3、启用中继日志

4、具有复制权限账号

5、解决自动增长列的问题

如果A服务器上自动增长的列编号有一个35，此时还没有同步到B服务器上，在B服务器上插入一条数据，编号也是35。当同步A的35到B服务器上来的话，必然产生数据丢失。

解决办法：让在A上插入的行的自动增长都为奇数，让B服务器上的自动增长都为偶数。这样就解决了自动增长的问题



## 三、流程

1. 在A、B服务器上创建具有复制权限的帐号

2. 在A、B服务器上修改配置文件（开启二进制日志、中继日志等）

3. 将A服务器上存在的数据文件导入到B服务器中（可选）

​          注意：导入数据的时候，先关闭B服务器的二进制日志。

4. 让B先成为slave，再让A成为slave

5. 测试



## 四、配置

2台主机分别是host111（192.168.255.100）和host222（192.168.255.200）

mysql安装过程略



1、创建授权用户

```bash
#host111执行

create user 'rep1'@'192.168.255.%' identified by 'mysql';
GRANT REPLICATION SLAVE ON *.* TO 'rep1'@'192.168.255.%';



#host222执行

create user 'rep1'@'192.168.255.%' identified by 'mysql';
GRANT REPLICATION SLAVE ON *.* TO 'rep1'@'192.168.255.%';
```





2、编辑配置文件

```bash
#host111配置

#vi /etc/my.cnf
[mysqld]
#
server-id = 111
port=3306
max_connections=1000
group_concat_max_len = 409600
log-bin = host111-bin
log-bin-index = host111-bin.index

#不需要复制的数据库
binlog-ignore-db = mysql
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
binlog-ignore-db = test
binlog_format = row

#日志文件过期天数
expire_logs_days = 7

pid-file=/usr/local/mysql/data/mysql.pid

#当每进行1次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

open_files_limit = 5010
table_open_cache = 2000 
max_allowed_packet = 32M
binlog_cache_size = 1M
max_heap_table_size = 16M
tmp_table_size = 16M
innodb_buffer_pool_size=2G
innodb_large_prefix=1
skip-name-resolve


# innodb
innodb_large_prefix=1

#排除不做复制的库
replicate-ignore-db = mysql
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
replicate-ignore-db = test
replicate-ignore-db = myopenfire
relay-log = host111-relay-bin
relay-log-index = host111-relay-bin.index

#如果A->B->C，其中B是A的从服务器，同时B又是C的主服务器，那么B服务器需要打开log-slave-updates选项
log-slave-updates = 1
read-only = 0
relay_log_recovery = 1

#自增步长
auto-increment-increment = 2
#起始序号
auto-increment-offset = 2

init-connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
tmpdir=/tmp
default-time-zone= system

symbolic-links=0

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/usr/local/mysql/mysql_error.log
```



```bash
#host 222配置
# cat /etc/my.cnf
[mysqld]
#
server-id = 222
port=3306
max_connections=1000
group_concat_max_len = 409600
log-bin = host222-bin
log-bin-index = host222-bin.index

#不需要复制的数据库
binlog-ignore-db = mysql
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
binlog-ignore-db = test
binlog_format = row

#日志文件过期天数
expire_logs_days = 7

pid-file=/usr/local/mysql/data/mysql.pid

#当每进行1次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

open_files_limit = 5010
table_open_cache = 2000 
max_allowed_packet = 32M
binlog_cache_size = 1M
max_heap_table_size = 16M
tmp_table_size = 16M
innodb_buffer_pool_size=2G
innodb_large_prefix=1
skip-name-resolve


# innodb
innodb_large_prefix=1

#排除不做复制的库
replicate-ignore-db = mysql
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
replicate-ignore-db = test
replicate-ignore-db = myopenfire
relay-log = host222-relay-bin
relay-log-index = host222-relay-bin.index

#如果A->B->C，其中B是A的从服务器，同时B又是C的主服务器，那么B服务器需要打开log-slave-updates选项
log-slave-updates = 1
read-only = 0
relay_log_recovery = 1

#自增步长
auto-increment-increment = 2
#起始序号
auto-increment-offset = 1

init-connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
tmpdir=/tmp
default-time-zone= system

symbolic-links=0

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/usr/local/mysql/mysql_error.log
```



3 、host111 查询状态

```sq
mysql> show master status;
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB                                 | Executed_Gtid_Set |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| host111-bin.000003 |      154 |              | mysql,information_schema,performance_schema,test |                   |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```

4、host222先配置成slave

```sql
#host222执行
mysql> change master to master_host='192.168.255.100',master_user='rep1',master_password='mysql',master_port=3306,MASTER_LOG_FILE='host111-bin.000003', MASTER_LOG_POS=154;
```



5、host222查询状态

```bash
mysql> show master status;
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB                                 | Executed_Gtid_Set |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| host222-bin.000002 |      154 |              | mysql,information_schema,performance_schema,test |                   |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```





5、再让host111配置成slave

```bash
#host111执行
mysql> change master to master_host='192.168.255.200',master_user='slave',master_password='mysql',master_port=3306,MASTER_LOG_FILE='host222-bin.000002', MASTER_LOG_POS=154;
```



6、启动复制

```bash
#启动从服务器复制功能 
Mysql>start slave;    #2台服务器都执行，host111启动从服务器复制功能，同理，host222也一样
```



7、查看状态

```bash
#host111状态
mysql> show master status;
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB                                 | Executed_Gtid_Set |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
| host111-bin.000004 |      154 |              | mysql,information_schema,performance_schema,test |                   |
+--------------------+----------+--------------+--------------------------------------------------+-------------------+
1 row in set (0.00 sec)

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.255.200
                  Master_User: rep1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: host222-bin.000003
          Read_Master_Log_Pos: 154
               Relay_Log_File: host111-relay-bin.000004
                Relay_Log_Pos: 371
        Relay_Master_Log_File: host222-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,test,myopenfire
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 748
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 222
                  Master_UUID: d3b07a7d-1594-11ea-b5e9-000c290cb27f
             Master_Info_File: /usr/local/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



```bash
#host222状态
mysql> show master status\G;
*************************** 1. row ***************************
             File: host222-bin.000003
         Position: 154
     Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql,information_schema,performance_schema,test
Executed_Gtid_Set: 
1 row in set (0.00 sec)

ERROR: 
No query specified

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.255.100
                  Master_User: rep1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: host111-bin.000004
          Read_Master_Log_Pos: 154
               Relay_Log_File: host222-relay-bin.000004
                Relay_Log_Pos: 371
        Relay_Master_Log_File: host111-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,test,myopenfire
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 748
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 111
                  Master_UUID: d1386990-1594-11ea-9cca-000c29abf8dc
             Master_Info_File: /usr/local/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```



## 五、测试

在111服务器上执行插入语句

```bash
mysql> create database prod;
Query OK, 1 row affected (0.00 sec)

mysql> use prod;
Database changed

mysql>  create table student(id int(10),name varchar(20));
Query OK, 0 rows affected (0.00 sec)

mysql> insert into student value ('111','aaa');
Query OK, 1 row affected (0.00 sec)

mysql> insert into student value('222','bbb');
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+------+------+
| id   | name |
+------+------+
|  111 | aaa  |
|  222 | bbb  |
+------+------+
2 rows in set (0.00 sec)
```

在222服务器上可以查询到记录

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| abc                |
| mysql              |
| performance_schema |
| prod               |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql> use prod;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_prod |
+----------------+
| student        |
+----------------+
1 row in set (0.00 sec)

mysql> select * from student;
+------+------+
| id   | name |
+------+------+
|  111 | aaa  |
|  222 | bbb  |
+------+------+
2 rows in set (0.00 sec)

```





在222服务器上执行插入

```bash
mysql> insert into student values('333','ccc');
Query OK, 1 row affected (0.00 sec)

mysql> insert into student values('444','ddd');
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+------+------+
| id   | name |
+------+------+
|  111 | aaa  |
|  222 | bbb  |
|  333 | ccc  |
|  444 | ddd  |
+------+------+
4 rows in set (0.00 sec)
```



 在111上也可以查询到记录

```bash
mysql> select * from student;
+------+------+
| id   | name |
+------+------+
|  111 | aaa  |
|  222 | bbb  |
|  333 | ccc  |
|  444 | ddd  |
+------+------+
4 rows in set (0.00 sec)
```

