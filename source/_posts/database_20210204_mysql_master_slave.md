---
layout:     post
title:      MYSQL Master - Slave 主从复制
date:       2021-02-04 17:05:13
author:     liangtong
catalog: true
categories: 数据库
tags: mysql

---

主从复制可以将数据可以从一个服务器数据库复制到其他服务器上。在复制数据时，一个服务器充当主服务器（master），其余的服务器充当从服务器（slave）。
Mysql服务器之间的主从同步是基于二进制日志机制，主服务器使用二进制日志来记录数据库的变动情况，从服务器通过读取和执行该日志文件来保持和主服务器的数据一致。
+ 读写分离，使数据库能支撑更大的并发
+ 发扬不同表引擎的优点（专用更新、查询服务器）
+ 热备


### 环境准备

```bash
mysql1(master): 172.16.30.72:3306
mysql2(slave): 172.16.xxxx.xxxx:xxxx (设置复制权限时，可不指定绝对ip)
```


### 主数据库

主数据库对应配置文件中（`mysqld.cnf`）需开启二进制日志机制，配置一个独立的 server_id ，关键配置内容如下：

```mysqld.cnf

[mysqld]
# bin log
server-id               = 100 #服务ID，用于区分服务，范围 1 ～ 2^32 -1
log_bin                 = /var/lib/mysql/mysql-bin # 日志文件名
expire_logs_days        = 10 #binlog过期清理时间
max_binlog_size         = 100M #binlog每个日志文件大小


sync_binlog = 1         #控制数据库的binlog刷到磁盘上去 , 0 不控制，性能最好，1每次事物提交都会刷到日志文件中，性能最差，最安全
binlog_format = mixed   #binlog日志格式，mysql默认采用statement，建议使用mixed
binlog_cache_size = 4m                        #binlog缓存大小
max_binlog_cache_size= 512m              #最大binlog缓存大
binlog-ignore-db=nacos_config,sys #不生成日志文件的数据库，多个数据库可以用逗号拼接，或者 复制这句话，写多行
# binlog-do-db=xxxx #可单独指定同步数据库，多个数据库可以用逗号拼接，或者 复制这句话，写多行

```

**修改完配置文件，需要重启主数据库**

```SQL
# 查看日志状态： true表示已生效
show variables like 'log_bin';
```

在主数据库上添加账户，并分配权限

```SQL
# 创建用户，设置密码
CREATE USER sync_slave IDENTIFIED BY 'sync_slave_psd';
# 分配用户复制的权限
grant replication slave on *.* to 'sync_slave'@'172.16.%'  identified by 'sync_slave_psd';

FLUSH PRIVILEGES;

```

操作完成后，可查看master状态

```SQL
# 查看master状态
show master status;
```

![](/post/db/20210204/master_state.png)


### 从数据库

从数据库对应配置文件中（`mysqld.cnf`）需要配置一个唯一的 server_id  ，关键配置内容如下：

```mysqld.cnf

[mysqld]
# bin log
server-id               = 101 #服务ID，用于区分服务，范围 1 ～ 2^32 -1


bind-address = 127.0.0.1
```


**修改完配置文件，需要重启从数据库**


配置从库

```SQL
# 配置从库，需要注意的是 master_log_file 和 master_log_pos 为 主数据库状态对应的 file 和 position 字段内容
change master to master_host='172.16.30.72',master_user='sync_slave',master_password='sync_slave_psd',master_log_file='mysql-bin.000005',master_log_pos=15128;
```

启动从库

```SQL
start slave;
```

查看是否成功

```SQL
show slave status; 
```

![](/post/db/20210204/slave_state.png)


### 测试

主数据库中，对表进行操作，查看从数据库对应表信息


### 其他

binlog_format 三种模式

① STATEMENT模式（SBR）

每一条会修改数据的sql语句会记录到binlog中。优点是并不需要记录每一条sql语句和每一行的数据变化，减少了binlog日志量，节约IO，提高性能。缺点是在某些情况下会导致master-slave中的数据不一致(如sleep()函数， last_insert_id()，以及user-defined functions(udf)等会出现问题)

② ROW模式（RBR）

不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了，修改成什么样了。而且不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题。缺点是会产生大量的日志，尤其是alter table的时候会让日志暴涨。

③ MIXED模式（MBR）

以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。

