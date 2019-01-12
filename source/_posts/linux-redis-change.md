---
title: linux_redis_change
categories: tech
tag: hide
date: 2018-12-19 00:17:16
tags:
---
```
#!/bin/sh
#监测Redis服务
masterProcess=`ps -ef|grep '/etc/redis.conf'|grep -v grep|wc -l`                #以主的方式启动的配置文件
failoverProcess=`ps -ef|grep '/etc/redis_failover.conf'|grep -v grep|wc -l`     #以从的方式启动的配置文件
if [ $masterProcess -eq 0 -a $failoverProcess -eq 0 ];then
     datetime=`date '+%Y%m%d%H:%M:%S'`
     /usr/local/bin/redis-cli -h 127.0.0.1 -p 6380 slaveof NO ONE    #将该Slave变为TempMaster
     mv /data/redisdb/dump.rdb /data/redisdb/$datetime".dump.rdb"    #原来的数据备份
     /usr/local/bin/redis-server /etc/redis_failover.conf            #使用另一个配置文件以从的方式启动，这样能自动同步最新数据
     sleep 2

     /usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 slaveof NO ONE    #变回以前的主
     sleep 2

     /usr/local/bin/redis-cli -h 127.0.0.1 -p 6380 slaveof 127.0.0.1 6379 #还原以前的主从
     sleep 2

     /usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 slaveof 127.0.0.1 6380 #实现双向同步
else



   `echo "process is ok!"
fi
```