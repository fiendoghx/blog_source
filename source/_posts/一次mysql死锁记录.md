---
title: 一次mysql死锁记录
categories:
  - Mysql
tags:
  - Mysql
  - 死锁
date: 2019-11-24 16:51:49
---

### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体过程](#3-具体过程)

## 1.缘由

由于一次数据库迁移，新的数据库以5.7.25版本安装，但以前的入库程序每次跑不了一天，一定会出现挂死的情况，排查后发现死锁，尝试了几种解决办法，做下记录

## 2.目的

提醒自己以后在使用Mysql时，要考虑到各个版本和锁机制的使用，避免出现死锁的情况

## 3.具体过程

Mysql版本: 5.7.25

发生死锁的表：
```
CREATE TABLE `table_example` (
  `dname` varchar(100) DEFAULT '',
  `subname` varchar(100) DEFAULT '',
  `data2` varchar(100) DEFAULT '',
  `data3` varchar(100) DEFAULT '',
  `capturetime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  KEY `subname` (`subname`),
  KEY `capturetime` (`capturetime`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
```

产生死锁的语句：
```
LOAD DATA LOCAL INFILE '/data/20191124/data.tmp'  INTO TABLE table_example 
FIELDS TERMINATED BY ';' (dname,subname,data2,data3,capturetime)
```

**怎么发现死锁的**

执行sql运行情况检索非sleep状态的sql
```
show OPEN TABLES where In_use > 0;
select id, db, user, host, command, time, state, info from 
information_schema.processlist where command != 'Sleep'  order by time desc;
```

检查到很多执行时间很长的sql

再执行Innodb死锁检测
```
show engine innodb status;
```

查到LATEST DETECTED DEADLOCK可以看到最近一次死锁记录

```
------------------------
LATEST DETECTED DEADLOCK
------------------------
2019-11-18 15:38:35 0x7fca119b8700
*** (1) TRANSACTION:
TRANSACTION 72276684, ACTIVE 0 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 18 lock struct(s), heap size 1136, 4 row lock(s)
MySQL thread id 813, OS thread handle 140505857419008, query id 46012 127.0.0.1 test executing
LOAD DATA LOCAL INFILE '/data/20191124/data.tmp'  INTO TABLE table_example 
FIELDS TERMINATED BY ';' (dname,subname,data2,data3,capturetime)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5819 page no 171968 n bits 200 index GEN_CLUST_INDEX of table `voltedata_volte`.`volte_rrd_data_20191118` trx id 72276684 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) TRANSACTION:
TRANSACTION 72276687, ACTIVE 0 sec inserting
mysql tables in use 1, locked 1
16 lock struct(s), heap size 1136, 4 row lock(s)
MySQL thread id 977, OS thread handle 140505855526656, query id 46014 127.0.0.1 test executing
LOAD DATA LOCAL INFILE '/data/20191124/data2.tmp'  INTO TABLE table_example 
FIELDS TERMINATED BY ';' (dname,subname,data2,data3,capturetime)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 5819 page no 171968 n bits 200 index GEN_CLUST_INDEX of table `voltedata_volte`.`volte_rrd_data_20191118` trx id 72276687 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5819 page no 171968 n bits 200 index GEN_CLUST_INDEX of table `voltedata_volte`.`volte_rrd_data_20191118` trx id 72276687 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;
```

**分析死锁原因**

从死锁记录能够看出，主要问题是两个事务相互等待GEN_CLUST_INDEX的S(共享)锁和IX(意向插入)锁导致死锁。

![](./img/article/dead_lock01.png)

由于S锁和IX锁是互斥的，所以发生申请IX锁时会进入队列，而两个事务申请先后不同，事务2先获取了S锁，所以事务1在申请IX锁时需要事务2释放S锁，而事务2又因为申请IX锁需要等待事务1先释放IX锁，形成了相互等待的情况，于是死锁。

**锁类型是否能共存**

| type | X | IX | S | IS |
| :----------:| :-----------:|  :-----------:|  :-----------:|  :-----------:| 
| X | × | × | × | × |
| IX | × | √ | × | √ |
| S | × | × | √ | √ |
| IS | × | √ | √ | √ |

GEN_CLUST_INDEX 是在对一张表进行update操作，没能找到对应唯一主键时，mysql生成的一张隐式索引表，我们这里遇到的就是这种情况，没有唯一主键。

**解决**

1.由于这张表平时并发入库并不多，主要是用作查询，最后把引擎换成MyISAM之后就解决了。

2.还有一种解决方式是降低了mysql版本到5.6.37。具体为什么降低到5.6.37之后没有死锁问题，没有具体深究。


