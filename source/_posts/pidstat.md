---
title: 进程实时监控pidstat命令详解
categories: tech
tag: hide
date: 2018-12-18 21:44:09
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

#网络传输质量越好，可以设定间隔时间越短，监控效果也越迅速;

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

<font color="red" size='10'>使用</font>
pidstat主要用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等。pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

实例讲解
默认参数

执行pidstat，将输出系统启动后所有活动进程的cpu统计信息：

复制代码
$ pidstat 1
Linux 3.13.0-49-generic (titanclusters-xxxxx) 07/14/2015 _x86_64_ (32 CPU)
07:41:02 PM UID PID %usr %system %guest %CPU CPU Command
07:41:03 PM 0 9 0.00 0.94 0.00 0.94 1 rcuos/0
07:41:03 PM 0 4214 5.66 5.66 0.00 11.32 15 mesos-slave
07:41:03 PM 0 4354 0.94 0.94 0.00 1.89 8 java
07:41:03 PM 0 6521 1596.23 1.89 0.00 1598.11 27 java
07:41:03 PM 0 6564 1571.70 7.55 0.00 1579.25 28 java
07:41:03 PM 60004 60154 0.94 4.72 0.00 5.66 9 pidstat
07:41:03 PM UID PID %usr %system %guest %CPU CPU Command
07:41:04 PM 0 4214 6.00 2.00 0.00 8.00 15 mesos-slave
07:41:04 PM 0 6521 1590.00 1.00 0.00 1591.00 27 java
07:41:04 PM 0 6564 1573.00 10.00 0.00 1583.00 28 java
07:41:04 PM 108 6718 1.00 0.00 0.00 1.00 0 snmp-pass
07:41:04 PM 60004 60154 1.00 4.00 0.00 5.00 9 pidstat
复制代码
pidstat命令输出进程的CPU占用率，该命令会持续输出，并且不会覆盖之前的数据，可以方便观察系统动态。如上的输出，可以看见两个JAVA进程占用了将近1600%的CPU时间，既消耗了大约16个CPU核心的运算资源。

指定采样周期和采样次数

pidstat命令指定采样周期和采样次数，命令形式为”pidstat [option] interval [count]”，以下pidstat输出以2秒为采样周期，输出10次cpu使用统计信息：

pidstat 2 10
cpu使用情况统计(-u)

使用-u选项，pidstat将显示各活动进程的cpu使用统计，执行”pidstat -u”与单独执行”pidstat”的效果一样。

 

内存使用情况统计(-r)

使用-r选项，pidstat将显示各活动进程的内存使用统计：

复制代码
linux:~ # pidstat -r -p 13084 1
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

15:08:18          PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
15:08:19        13084 133835.00      0.00 15720284 15716896  96.26  mmmm
15:08:20        13084  35807.00      0.00 15863504 15849756  97.07  mmmm
15:08:21        13084  19273.87      0.00 15949040 15792944  96.72  mmmm
复制代码
以上各列输出的含义如下：

minflt/s: 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数
majflt/s: 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，这样的page fault为major page fault，一般在内存使用紧张时产生
VSZ:      该进程使用的虚拟内存(以kB为单位)
RSS:      该进程使用的物理内存(以kB为单位)
%MEM:     该进程使用内存的百分比
Command:  拉起进程对应的命令
IO情况统计(-d)


使用-d选项，我们可以查看进程IO的统计信息：

复制代码
linux:~ # pidstat -d 1 2
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

17:11:36          PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
17:11:37        14579 124988.24      0.00      0.00  dd

17:11:37          PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
17:11:38        14579 105441.58      0.00      0.00  dd
复制代码
输出信息含义

kB_rd/s: 每秒进程从磁盘读取的数据量(以kB为单位)
kB_wr/s: 每秒进程向磁盘写的数据量(以kB为单位)
Command: 拉起进程对应的命令
针对特定进程统计(-p)

使用-p选项，我们可以查看特定进程的系统资源使用情况：

复制代码
linux:~ # pidstat -r -p 1 1
Linux 2.6.32.12-0.7-default (linux)             06/18/12        _x86_64_

18:26:17          PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
18:26:18            1      0.00      0.00   10380    640   0.00  init
18:26:19            1      0.00      0.00   10380    640   0.00  init
……
复制代码
pidstat常用命令
使用pidstat进行问题定位时，以下命令常被用到：

pidstat -u 1

pidstat -r 1

pidstat -d 1
以上命令以1秒为信息采集周期，分别获取cpu、内存和磁盘IO的统计信息。