---
layout:     post
title:      FMDB
date:       2017-04-27 12:07:43
author:     liangtong
catalog: true
categories: 数据库
tags: SQLite3

---

* content
{:toc}

​	SQLite是一款轻型的数据库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。它使用B+树存储表，整个SQLite数据库就是这些B+树组成的森林。。它的设计目标是嵌入式的，目前被广泛应用。<a href="https://github.com/ccgus/fmdb/"> FMDB </a> 是基于SQLite的实现 。




### 数据库（连接）   
数据库在使用前均需要建立连接，FMDB提供了两种方式对应数据库文件’通常xx.db’路径建立连接。FMDatabase是一个单一的用来执行SQL语句的数据库连接，使用同步的方式进行，如果需要在多线程执行查询或者更新，则需要使用FMDatabaseQueue

#### FMDatabase      
FMDatabase可以用来执行增、删、改、查相关的SQL语句，通常情况下，不会有什么问题。这里需要说明的一点是移动端在涉及到多线程操作时，不要使用单例模式建立一个FMDatabase对象，否则会发生死锁问题。其官方文档中也有给出警告: Do not instantiate a single `FMDatabase` object and use it across multiple threads.

#### FMDatabaseQueue    
FMDatabaseQueue的方法调用时异步的，通常在多线程中使用。即使在不同的闭包中使用，也是在发送在同一个线程中。

### 表结构操作      
SQLite对于每个表的元数据(表名、根节点地址、表scheme等)信息均记录在一个叫’sql_master’的表中。常见的表结构变更包括创建、修改、删除，这些信息的变更均会影响’sql_master’表。

#### 创建      
表的创建跟通常sql语句区别不大，通常在表创建时设置主键、默认值等。使用语法如下：   
```SQL
CREATE TABLE  IF NOT EXISTS RecordSign(recordId text NOT NULL PRIMARY KEY, 
					projectName TEXT, 
					visibleState INTEGER DEFAULT 0 , 
					lastTriggerTime text DEFAULT '2017-03-13 23:59:59')
```

#### 查询     
判断某个表是否存在，语法如下：   
```SQL
select * from sqlite_master where type = 'table' and name = '表名'
```

判断某个表中是否存在某列，语法如下（通过判断sql语句中是否包含列名）：   
```SQL
select * from sqlite_master where type = 'table' and name = '表名' and sql like %列表%
```

#### 删除    
表的删除，使用drop语句：  
```SQL
DROP TABLE '表名'
```

### 表内容操作
多表联合操作使用关键字：`join`，或者`in`(如果不需要跨表取字段的话)，比如：       
```SQL
NSString* filterBusinessIDSql = [NSString stringWithFormat:@"select businessID from BeaconRelatedPolicyAndBusiness where isValid = 1 and policyID in ('%@')", [minPriorityPolicyIDArray componentsJoinedByString:@"','"]];  

NSString* retSql = [NSString stringWithFormat:@"select * from BeaconBusinessMsg where businessID in (%@) and isValid = 1 order by priority asc",filterBusinessIDSql];
```

#### 插入   
支持使用`insert` 或 `insert or replace` 或 `insert or ignore` 等语法   
```SQL
[dbms executeUpdate:@"insert OR IGNORE into RecordSign (recordId, projectName,visibleState) values (?, ?,  1)",
                         recordId,
                         projectName
                         ];
```
#### 更新     
使用`update`语句进行表结构的更新，如下：    
```SQL
[dbms executeUpdate:@"update RecordSign set projectName = ? ,  visibleState = 1 where  recordId = ? ",
               projectName,
               recordId
				];
```
#### 查询    
使用 `select` 关键字进行表数据查询，返回 `ResultSet` ，然后可以根据具体情况进行处理。     
```SQL
[dbms executeQuery:@"select * from RecordSign where  recordId = ? ",recordId];
```
#### 删除    
使用 `delete` 关键字来删除表中的记录。    
```SQL
 [dbms executeUpdate:@"delete from RecordSign where visibleState = 0"];
```
#### 批处理      
当涉及到数据量比较大或者原子性操作时，可能会用到批处理相关操作，例如：    
```SQL
//假设数据量不是太大，不需要通过分批次提交
[[GKDatabaseManager sharedInstance].dbQueue inTransaction:^(FMDatabase *dbms, BOOL *rollback) {
    [dbms executeUpdate:@"update  RecordSign set visibleState = 0 "];
    for (NSDictionary* projectModel in projectList) {
        NSString* recordId = [projectModel objectForKey:@"recordId"];
        NSString* projectName = [projectModel objectForKey:@"projectName"];

        FMResultSet* retSet = [dbms executeQuery:@"select * from RecordSign where  recordId = ? ",recordId];
        if ([retSet next]) {
            [dbms executeUpdate:@"update RecordSign set projectName = ? ,  visibleState = 1 where  recordId = ? ",
                        projectName,
                         recordId
                         ];
        }else{
            [dbms executeUpdate:@"insert OR IGNORE into RecordSign (recordId, projectName,visibleState) values (?, ?,  1)",
                         recordId,
                         projectName
                         ];
        }//end if retSet
        [retSet close];
    }//end for
    [dbms executeUpdate:@"delete from RecordSign where visibleState = 0"];
}];
[[GKDatabaseManager sharedInstance].dbQueue close];
```

### 使用心结     

我的态度是：数据库能实现的东西，自己就没必要再折腾了。数据库本身是一个很强大的软件，她已经可以帮助我们解决很多的问题。
所以在拿到需求后，别急着写代码逻辑，最好先考虑下是否可以利用这些现成的工具。有时候数据库处理数据的能力比自己写程序效率要高很多，且代价也比较小；数据库已经实现的功能，当然没有必要再自行编写程序解决，除非你觉得自己比数据库处理的好。      



- 使用SQLite数据库处理缓存的话，需要考虑数据库表结构的更新。
    - 建议在建立表结构的时候，添加版本信息，比如 `tablename_v1`。就像CoreData处理版本升级时的version版本号一样。
    - 数据迁移。
    - 表结构受损后的数据恢复。   
- 数据库连接的选择
    - FMDatabase是同步操作，FMDatabaseQueue是异步操作。
    - 不要单例FMDatabase对象，在多线程处理时各种死锁绝对够你受的。
    - 不要将FMDatabase和FMDatabaseQueue同时使用，否则等着数据异常或者死锁崩溃吧。



