---
title: zabbix
categories: linux
tag: hide
date: 2018-12-18 21:37:44
tags:
---

<font color="red" size='10'>安装</font>

```autoyum -y install curl curl-devel net-snmp net-snmp-devel perl-DBI php-gd php-xml php-bcmath libxml2-devel
./configure --prefix=/usr/local/zabbix_3.0.8 \
--enable-server \
--enable-proxy \
--enable-agent \
--with-mysql=/usr/bin/mysql_config \
--with-net-snmp \
--with-libcurl --with-net-snmp --with-libcurl --with-libxml2

groupadd zabbix
useradd zabbix -g zabbix -s /bin/false

create database zabbix character set utf8;
grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';

mysql -uzabbix -pzabbix -hlocalhost zabbix < database/mysql/schema.sql
mysql -uzabbix -pzabbix -hlocalhost zabbix < database/mysql/images.sql
mysql -uzabbix -pzabbix -hlocalhost zabbix < database/mysql/data.sql

cd /usr/local && wget 192.168.1.64/zabbix.tar.gz && tar fxz zabbix.tar.gz && cd zabbix_2.2.3 && ./setup_agentd.sh
cd /usr/local && wget 192.168.3.104/zabbix.tar.gz && tar fxz zabbix.tar.gz && cd zabbix_2.2.3 && ./setup_agentd.sh

cd /usr/local/ && wget 192.168.3.105/bi/zabbix-2.2.7.tar &&tar fx zabbix-2.2.7.tar && cd zabbix-2.2.7 && ./zabbix_setup.sh

pid = run/php-fpm.pid
error_log = log/php-fpm.log
log_level = notice
emergency_restart_threshold = 60
emergency_restart_interval = 60s
process_control_timeout = 3
daemonize = yes
pm = dynamic
pm.max_children = 500
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 30


pm.max_requests = 1000
pm.status_path = /status
ping.path = /ping
ping.response = pong
request_terminate_timeout = 8
request_slowlog_timeout = 10s
slowlog = log/$pool.log.slow
rlimit_files = 1024
rlimit_core = 0
chroot =
chdir =
catch_workers_output = yes


Server=zabbix_server #zabbix server的ip地址或者域名

Hostname=Telcom_proxy #proxy主机名，在zabbix web会以这个名字为准

#DB 设定档

DBName=zabbix

DBUser=zabbix

DBPassword=zabbixpass

ProxyLocalBuffer=0 #设定为0小时，除非有其他第三方应用和插件需要调用

ProxyOfflineBuffer=1 #proxy或者server无法连接时，保留离线的监控数据的时间，单位小时

ConfigFrequency=600 #server和proxy配置修改同步时间间隔，设定5-10分钟即可。

DataSenderFrequency=10 #数据发送时间间隔，10-30s;

网络传输质量越好，可以设定间隔时间越短，监控效果也越迅速;

StartPollers=10 #开启多线程数，一般不要超过30个;

StartPollersUnreachable=1 #该线程用来单独监控无法连接的主机，1个即可;

StartTrappers=10 #trapper线程数

StartPingers=1 #fping线程数

CacheSize=64M #用来保存监控数据的缓存数，根据监控主机数量适当调整;

Timeout=10 #超时时间，设定不要超过30s，不然会拖慢其他监控数据抓取时间;

TrapperTimeout=30 #同上

FpingLocation=/usr/sbin/fping #配合simple check icmp检测使用，如不需要可关闭


echo "UserParameter=redis_stats[*],/usr/local/redis/bin/redis-cli -a ucenter -h `grep 192.168 /etc/sysconfig/network-scripts/ifcfg-em2 |awk -F= '{print $2}'` -p \$1 info|grep \$2|cut -d : -f2">>/usr/local/zabbix_2.2.3/etc/zabbix_agentd.conf

echo "UserParameter=redis_stats[*],/usr/local/redis/bin/redis-cli -h `grep 192.168 /etc/sysconfig/network-scripts/ifcfg-em2 |awk -F= '{print $2}'` -p \$1 info|grep \$2|cut -d : -f2">>/usr/local/zabbix_2.2.3/etc/zabbix_agentd.conf

sed 's/UserParameter.*//g' /usr/local/zabbix_2.2.3/etc/zabbix_agentd.conf
停留在安装页面执行
mkdir /var/lib/php/session
chmod 777 /var/lib/php/sessionauto
```

<font color="red" size='10'>zabbix中常用到的几个key</font>

* 1、监控端口的：net.tcp.port[,3306]
* /usr/local/zabbix/bin/ -s192.168.8.120 -knet.tcp.port[,3306] 返回1为192.168.8.120的端口3306存在，0为不存在
* 2、监控进程的：proc.num[mysqld]
* /usr/local/zabbix/bin/zabbix_get -s192.168.8.120 -kproc.num[mysqld] 返回值为192.168.8.120中mysqld的进程数量
* /usr/local/zabbix/bin/zabbix_get -s192.168.8.120 -kproc.num[] 返回值为192.168.8.120中所有的进程数量
* 3、查看CPU核数的：system.cpu.num 返回值为服务器CPU的核数
* 4、查看系统的系统启动时间和当前时间：system.boottime、system.localtime 返回值为系统启动时间和当前时间，为时间戳格式
* 5、查看系统的简单信息：system.uname 返回值为192.168.8.120的系统信息，类似于linux系统的uname -a命令
* 6、查看windowns系统当前网卡的进出流量：net.if.out[{HOST.NAME},bytes]、net.if.in[{HOST.NAME},bytes]和linux系统的key：net.if.out[eth0,bytes]、net.if.in[eth0,bytes]一样
* /usr/local/zabbix/bin/zabbix_get -s192.168.8.120 -knet.if.in[192.168.8.120,bytes] 返回值为IP为192.168.8.120的进流量，此值为计数值，单位为bytes，减去上次取得值，除以时间间隔为此段时间内的平均流量
* /usr/local/zabbix/bin/zabbix_get -s192.168.8.120 -knet.if.out[192.168.8.120,bytes] 返回值为IP为192.168.8.120的出流量，此值为计数值，单位为bytes，减去上次取得值，除以时间间隔为此段时间内的平均流量
* 7、查看系统内存大小：vm.memory.size[total]，返回值单位bytes
* 8、查看文件的大小： vfs.file.size[file] 如： vfs.file.size[/var/log/syslog] 返回的是/var/log/syslog的大小，单位是：bytes
* 9、查看文件是否存在：vfs.file.exists[file] 文件如果存在返回0，不存在返回1
* 10、查看文件的MD5：vfs.file.md5sum[file]查看小文件的MD5，返回为MD5值(好像只有2.0以上的版本有这个key)
* 11、自动发现网卡并监控流量和自动发现分区及分区挂载情况的两个key：net.if.discovery，vfs.fs.discovery，windows和linux监控模板中都有这模板(2.0以上版本)，应用即可

以上是常用的key，其实监控服务器无非就是内存、硬盘占用、CPU负载、流量、服务器和端口等情况。如果要监控其他的可以自定义key来实现，本人喜欢自定义key，写个脚本来返回，得到自己想要的监控结果，zabbix在这块做的非常好，扩展性很强，支持各种脚本来实现自定义的key。

要启用自定义key，需要在客户端的配置文件中启用UnsafeUserParameters=1参数，然后在配置文件的最下面来定义key，如：
UserParameter=free.disk,/usr/local/zabbix/bin/disk.py
free.disk为key的名字，/usr/local/zabbix/bin/disk.py为服务器端调用free.disk这个key时执行的脚本，其结果就是free.disk的返回值。脚本可以是任何可以运行的脚本语言。

```auto监控io
UserParameter=custom.vfs.dev.read.ops[*],cat /proc/diskstats | grep $1 | head -1 |awk '{print $$4}'
UserParameter=custom.vfs.dev.read.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$7}'
UserParameter=custom.vfs.dev.write.ops[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$8}'
UserParameter=custom.vfs.dev.write.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$11}'
UserParameter=custom.vfs.dev.io.active[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$12}'
UserParameter=custom.vfs.dev.io.ms[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$13}'
UserParameter=custom.vfs.dev.read.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$6}'
UserParameter=custom.vfs.dev.write.sectors[*],cat /proc/diskstats | grep $1 | head -1 | awk '{print $$10}'auto```