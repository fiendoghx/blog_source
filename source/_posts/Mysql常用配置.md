---
title: Mysql常用配置
categories:
  - Mysql
tags:
  - 配置
date: 2019-11-18 11:33:03
---

### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体实施](#3-具体实施)
    * [3.1 事前准备](#3-1事前准备)
    * [3.2 实施并验证](#3-2实施并验证)
* [4.为什么采用以上方式](#4-为什么采用以上方式)

## 1.缘由



## 2.目的



## 3.具体实施



### 3.1事前准备



### 3.2实施并验证

**具体配置**

```
[client]
port=3306
socket=/usr/local/var/mysql.sock
default-character-set=utf8

[mysqld]
user=mysql
bind-address=0.0.0.0
port=3306
datadir=/data/mysql/
socket=/usr/local/var/mysql.sock
log_bin=/data/mysql/mysql-bin
expire_logs_days=7
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

max-connections=2048
max_connect_errors=500
max_allowed_packet=16M
open-files-limit=65536

character-set-server=utf8
default-storage-engine=InnoDB
lower_case_table_names=1
sql_mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION

server-id=1

innodb_data_home_dir=/data/mysql/
innodb_file_per_table=1
innodb_open_files=4096
table_open_cache=4096

#Forced autoinc locking
binlog_format=row
innodb_autoinc_lock_mode=2

#load data
innodb_buffer_pool_size=62G


innodb_log_buffer_size=16M
innodb_log_file_size=256M
innodb_log_files_in_group=8


[mysqld_safe]
log_bin=/data/mysql/mysql-bin
expire_logs_days=7
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```

## 4.为什么采用以上方式



