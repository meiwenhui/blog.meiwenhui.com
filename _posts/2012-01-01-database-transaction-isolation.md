---
layout: post
title: "数据库的四种隔离级别及spring事务(Transaction)的七种事务传播行为"
description: "数据库的五种隔离级别及spring事务(Transaction)的七种事务传播行为"
keywords: "数据库, 事务, 隔离级别"
categories: [database]
permalink: /:categories/isolation/
---
### 什么是事务
事务，就是一组操作数据库的动作集合。事务是现代数据库理论中的核心概念之一。   
如果一组处理步骤或者全部发生或者一步也不执行，我们称该组处理步骤为一个事务。  
当所有的步骤像一个操作一样被完整地执行，我们称该事务被提交。  
由于其中的一部分或多步执行失败，导致没有步骤被提交，则事务必须回滚到最初的系统状态。  

#### 四个隔离级别：

**isolation_read_uncommitted**  
这是事务最低的隔离级别，它充许**别外一个事务**可以看到**这个事务未提交**的数据。
这种隔离级别会产生脏读，不可重复读和幻像读。

**isolation_read_committed**  
保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。
这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。

**isolation_repeatable_read**  
这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。
它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。

**isolation_serializable**  
这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。
除了防止脏读，不可重复读外，还避免了幻像读。
在A客户端中
```sql
mysql> set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from account;
+------+--------+---------+
| id   | name   | balance |
+------+--------+---------+
|    1 | lilei  |   10000 |
|    2 | hanmei |   10000 |
|    3 | lucy   |   10000 |
|    4 | lily   |   10000 |
+------+--------+---------+
rows in set (0.00 sec)
```
打开一个客户端B，并设置当前事务模式为serializable，插入一条记录报错，表被锁了插入失败，mysql中事务隔离级别为serializable时会锁表，因此不会出现幻读的情况，这种隔离级别并发性极低，开发中很少会用到。
```sql
mysql> set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into account values(5,'tom',0);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

###### 关键词：  
1)幻读：在一个事务内读取了别的事务插入的数据，导致前后读取不一致(insert)   
事务1读取记录时事务2增加了记录并提交，事务1再次读取时可以看到事务2新增的记录；  

2)脏读：指一个事务读取了一个未提交事务的数据   
事务1更新了记录，但没有提交，事务2读取了更新后的行，然后事务T1回滚，现在T2读取无效。  

3)不可重复读取：在一个事务内读取表中的某一行数据,多次读取结果不同.一个事务读取到了另一个事务提交后的数据.  
事务1读取记录时，事务2更新了记录并提交，事务1再次读取时看到的是事务2修改后的记录；



###### 补充：   
1、事务隔离级别为读提交时，写数据只会锁住相应的行   
2、事务隔离级别为可重复读时，如果检索条件有索引（包括主键索引）的时候，默认加锁方式是next-key 锁；如果检索条件没有索引，更新数据时会锁住整张表。一个间隙被事务加了锁，其他事务是不能在这个间隙插入记录的，这样可以防止幻读。   
3、事务隔离级别为串行化时，读写数据都会锁住整张表   
4、隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。  


https://www.cnblogs.com/huanongying/p/7021555.html
