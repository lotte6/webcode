---
title: mysql_configure
categories: mysql
tag: hide
date: 2019-01-04 15:49:03
tags:
---

mysql 编译安装


```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EXTRA_CHARSETS:STRING=utf8,gbk \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_DATADIR=/var/mysql/data \
-DWITH_EMBEDDED_SERVER=1  \
-DEXTRA_CHARSETS=all \
-DMYSQL_USER=mysql


DCMAKE_INSTALL_PREFIX=/usr/local/mysql #mysql安装的主目录，默认为/usr/local/mysql
DMYSQL_DATADIR=/usr/local/mysql/data #mysql数据库文件的存放目录，可以自定义
DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock #系统Socket文件（.sock）设置,基于该文件路径进行Socket链接，必须为绝对路径
DSYSCONFDIR=/etc #mysql配置文件 my.cnf的存放地址，默认为/etc下
DMYSQL_TCP_PORT=3306 #数据库服务器监听端口，默认为3306
DENABLED_LOCAL_INFILE=1 #允许从本地导入数据
DWITH_READLINE=1 #快捷键功能
DWITH_SSL=yes #支持 SSL
DMYSQL_USER=mysql #默认为mysql

//下面3个是数据库编码设置
DEXTRA_CHARSETS=all #安装所有扩展字符集，默认为all
DDEFAULT_CHARSET=utf8 #使用 utf8 字符
DDEFAULT_COLLATION=utf8_general_ci #校验字符

//下面5个是数据库存储引擎设在
DWITH_MYISAM_STORAGE_ENGINE=1 #安装 myisam 存储引擎
DWITH_INNOBASE_STORAGE_ENGINE=1 #安装 innodb 存储引擎
DWITH_ARCHIVE_STORAGE_ENGINE=1 #安装 archive 存储引擎
DWITH_BLACKHOLE_STORAGE_ENGINE=1 #安装 blackhole 存储引擎
DWITH_PARTITION_STORAGE_ENGINE=1 #安装数据库分区
```
