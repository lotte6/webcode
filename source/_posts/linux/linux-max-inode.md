---
title: linux 文件描述符
categories: linux
tag: hide
date: 2019-01-23 12:08:13
tags:
---
/proc/sys/fs/file-max

/proc/sys/fs/file-nr 其中第一个数表示当前系统已分配使用的打开文件描述符数，第二个数为分配后已释放的（目前已不再使用），第三个数等于file-max。
/proc/sys/kernel/shmall 该文件是在任何给定时刻系统上可以使用的共享内存的总量（以字节为单位）。

/proc/sys/kernel/shmax 该文件指定内核所允许的最大共享内存段的大小（以字节为单位）。

/proc/sys/kernel/shmmni 该文件表示用于整个系统共享内存段的最大数目

所有进程打开的文件描述符数不能超过/proc/sys/fs/file-max  
单个进程打开的文件描述符数不能超过user limit中nofile的soft limit  
nofile的soft limit不能超过其hard limit  
nofile的hard limit不能超过/proc/sys/fs/nr_open  

一般会修改两个文件，/etc/sysctl.conf和/etc/security/limits.conf， 用来配置TCP/IP参数和最大文件描述符。
TCP/IP参数配置
修改文件/etc/sysctl.conf,配置网络参数。
```
net.ipv4.tcp_wmem = 4096 87380 4161536
net.ipv4.tcp_rmem = 4096 87380 4161536
net.ipv4.tcp_mem = 786432 2097152 3145728
```
数值根据需求进行调整。更多的参数可以看以前整理的一篇文章: Linux TCP/IP 协议栈调优 。
执行/sbin/sysctl -p即时生效。  

最大文件描述符  
Linux内核本身有文件描述符最大值的**，你可以根据需要更改：
```
系统最大打开文件描述符数：/proc/sys/fs/file-max
临时性设置：echo 1000000 > /proc/sys/fs/file-max
永久设置：修改/etc/sysctl.conf文件，增加fs.file-max = 1000000
进程最大打开文件描述符数
使用ulimit -n查看当前设置。使用ulimit -n 1000000进行临时性设置。
要想永久生效，你可以修改/etc/security/limits.conf文件，增加下面的行：

* hard nofile 1000000
* soft nofile 1000000
root hard nofile 1000000
root soft nofile 1000000
```
还有一点要注意的就是hard limit不能大于/proc/sys/fs/nr_open，因此有时你也需要修改nr_open的值。
```
执行echo 2000000 > /proc/sys/fs/nr_open
```
查看当前系统使用的打开文件描述符数，可以使用下面的命令：
```
cat /proc/sys/fs/file-nr
1632 0 1513506
其中第一个数表示当前系统已分配使用的打开文件描述符数，第二个数为分配后已释放的（目前已不再使用），第三个数等于file-max。
```

file-max的含义

```
man proc，可得到file-max的描述：
/proc/sys/fs/file-max
              This  file defines a system-wide limit on the number of open files for all processes.  (See
              also setrlimit(2),  which  can  be  used  by  a  process  to  set  the  per-process  limit,
              RLIMIT_NOFILE,  on  the  number  of  files it may open.)  If you get lots of error messages
              about running out of file handles, try increasing this value:

即file-max是设置 系统所有进程一共可以打开的文件数量 。同时一些程序可以通过setrlimit调用，设置每个进程的限制。如果得到大量使用完文件句柄的错误信息，是应该增加这个值。

echo  6553560 > /proc/sys/fs/file-max
或修改 /etc/sysctl.conf, 加入
fs.file-max = 6553560 重启生效
```