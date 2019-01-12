---
title: linux_inotify
categories: tech
tag: hide
date: 2018-12-18 23:49:48
tags:
---

* 在前面的博文中，我讲到过利用rsync实现数据的镜像和备份，但是要实现数据的实时备份，单独靠rsync还不能实现，本文就讲述下如何实现数据的实时备份。
* 一、rsync的优点与不足
 * 与传统的cp、tar备份方式相比，rsync具有安全性高、备份迅速、支持增量备份等优点，通过rsync可以解决对实时性要求不高的数据备份需求，例如定期的备份文件服务器数据到远端服务器，对本地磁盘定期做数据镜像等。
 * 随着应用系统规模的不断扩大，对数据的安全性和可靠性也提出的更好的要求，rsync在高端业务系统中也逐渐暴露出了很多不足，首先，rsync同步数据时，需要扫描所有文件后进行比对，进行差量传输。如果文件数量达到了百万甚至千万量级，扫描所有文件将是非常耗时的。而且正在发生变化的往往是其中很少的一部分，这是非常低效的方式。其次，rsync不能实时的去监测、同步数据，虽然它可以通过linux守护进程的方式进行触发同步，但是两次触发动作一定会有时间差，这样就导致了服务端和客户端数据可能出现不一致，无法在应用故障时完全的恢复数据。基于以上原因，rsync+inotify组合出现了！
* 二、 初识inotify
 * Inotify 是一种强大的、细粒度的、异步的文件系统事件监控机制，linux内核从2.6.13起，加入了Inotify支持，通过Inotify可以监控文件系统中添加、删除，修改、移动等各种细微事件，利用这个内核接口，第三方软件就可以监控文件系统下文件的各种变化情况，而inotify-tools就是这样的一个第三方软件。
* 在上面章节中，我们讲到，rsync可以实现触发式的文件同步，但是通过crontab守护进程方式进行触发，同步的数据和实际数据会有差异，而inotify可以监控文件系统的各种变化，当文件有任何变动时，就触发rsync同步，这样刚好解决了同步数据的实时性问题。
* 三、 安装inotify工具inotify-tools
 * 由于inotify特性需要Linux内核的支持，在安装inotify-tools前要先确认Linux系统内核是否达到了2.6.13以上，如果Linux内核低于2.6.13版本，就需要重新编译内核加入inotify的支持，也可以用如下方法判断，内核是否支持inotify：
* [root@localhost webdata]# uname -r
* 2.6.18-164.11.1.el5PAE
* [root@localhost webdata]# ll /proc/sys/fs/inotify
* 总计 0
* -rw-r--r-- 1 root root 0 04-13 19:56 max_queued_events
* -rw-r--r-- 1 root root 0 04-13 19:56 max_user_instances
* -rw-r--r-- 1 root root 0 04-13 19:56 max_user_watches
* 如果有上面三项输出，表示系统已经默认支持inotify，接着就可以开始安装inotify-tools了。
* 可以到http://inotify-tools.sourceforge.net/下载相应的inotify-tools版本，然后开始编译安装：
* [root@localhost  ~]# tar zxvf inotify-tools-3.14.tar.gz 
* root@localhost  ~]# cd inotify-tools-3.14
* [root@localhost inotify-tools-3.14]# ./configure
* [root@localhost inotify-tools-3.14]# make
* [root@localhost inotify-tools-3.14]# make install
* [root@localhost inotify-tools-3.14]# ll /usr/local/bin/inotifywa*
* -rwxr-xr-x 1 root root 37264 04-14 13:42 /usr/local/bin/inotifywait
* -rwxr-xr-x 1 root root 35438 04-14 13:42 /usr/local/bin/inotifywatch
* inotify-tools安装完成后，会生成inotifywait和inotifywatch两个指令，其中，inotifywait用于等待文件或文件集上的一个特定事件，它可以监控任何文件和目录设置，并且可以递归地监控整个目录树。
* inotifywatch用于收集被监控的文件系统统计数据，包括每个inotify事件发生多少次等信息。
* 四、 inotify相关参数
* inotify定义了下列的接口参数，可以用来限制inotify消耗kernel memory的大小。由于这些参数都是内存参数，因此，可以根据应用需求，实时的调节其大小。下面分别做简单介绍。
    * /proc/sys/fs/inotify/max_queued_evnets     
       * 表示调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值的事件被丢弃，但会触发IN_Q_OVERFLOW事件。
    * /proc/sys/fs/inotify/max_user_instances
        * 表示每一个real user ID可创建的inotify instatnces的数量上限。
    * /proc/sys/fs/inotify/max_user_watches
        * 表示每个inotify instatnces可监控的最大目录数量。如果监控的文件数目巨大，需要根据情况，适当增加此值的大小，例如：
* echo 30000000 > /proc/sys/fs/inotify/max_user_watches
* 五、 inotifywait相关参数
* Inotifywait是一个监控等待事件，可以配合shell脚本使用它，下面介绍一下常用的一些参数：
*  -m， 即--monitor，表示始终保持事件监听状态。
*  -r， 即--recursive，表示递归查询目录。
*  -q， 即--quiet，表示打印出监控事件。
*  -e， 即--event，通过此参数可以指定要监控的事件，常见的事件有modify、delete、create、attrib等。
* 更详细的请参看man  inotifywait。
* 六、 rsync+inotify企业应用案例
 * 案例描述
* 这是一个CMS内容发布系统，后端采用负载均衡集群部署方案，有一个负载调度节点和三个服务节点以及一个内容发布节点构成，内容发布节点负责将用户发布的数据生成静态页面，同时将静态网页传输到三台服务节点，而负载调度节点负责将用户请求根据负载算法调度到相应的服务节点，实现用户访问。用户要求在前端访问到的网页数据始终是最新的、一致的。
* 解决方案
* 为了保证用户访问到的数据一致性和实时性，必须保证三个服务节点与内容发布节点的数据始终是一致的，这就需要通过文件同步工具来实现，这里采用rsync，同时又要保证数据是实时的，这就需要inotify，即：使用inotify监视内容发布节点文件的变化，如果文件有变动，那么就启动rsync，将文件实时同步到三个服务节点。
* 系统环境
* 这里所有服务器均采用Linux操作系统，系统内核版本与节点信息如表1 所示：
* 表1

* 1 安装rsync与inotify-tools
* inotify-tools是用来监控文件系统变化的工具，因此必须安装在内容发布节点，服务节点无需安装inotify-tools，另外需要在web1、web2、web3、webserver节点上安装rsync，由于安装非常简单，这里不在讲述。
* 在这个案例中，内容发布节点（即server）充当了rsync客户端的角色，而三个服务节点充当了rsync服务器端的角色，整个数据同步的过程，其实就是一个从客户端向服务端推送数据的过程。这点与上面我们讲述的案例刚好相反。
* 2 在三个服务节点配置rsync
 * 这里给出三个服务节点的rsync配置文件，以供参考，读者可根据实际情况自行修改。
* Web1节点rsyncd.conf配置如下：
```
uid = nobody
gid = nobody
use chroot = no
max connections = 10
strict modes = yes
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[web1]
path = /web1/wwwroot/
comment = web1 file
ignore errors
read only = no
write only = no
hosts allow = 192.168.12.134
hosts deny = *
list = false
uid = root
gid = root
auth users = web1user
secrets file = /etc/web1.pass
Web2节点rsyncd.conf配置如下：
uid = nobody
gid = nobody
use chroot = no
max connections = 10
strict modes = yes
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[web2]
path = /web2/wwwroot/
comment = web2 file
ignore errors
read only = no
write only = no
hosts allow = 192.168.12.134
hosts deny = *
list = false
uid = root
gid = root
auth users = web2user
secrets file = /etc/web2.pass
Web3节点rsyncd.conf配置如下：
uid = nobody
gid = nobody
use chroot = no
max connections = 10
strict modes = yes
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[web3]
path = /web3/wwwroot/
comment = web3 file
ignore errors
read only = no
write only = no
hosts allow = 192.168.12.134
hosts deny = *
list = false
uid = root
gid = root
auth users = web3user
secrets file = /etc/web3.pass
在三台服务节点rsyncd.conf文件配置完成后，依次启动rsync守护进程，接着将rsync服务加入到自启动文件中：
echo  “/usr/local/bin/rsync --daemon” >>/etc/rc.local
到此为止，三个web服务节点已经配置完成。
3 配置内容发布节点
 配置内容发布节点的主要工作是将生成的静态网页实时的同步到集群中三个服务节点，这个过程可以通过一个shell脚本来完成，脚本内容大致如下：
#!/bin/bash
host1=192.168.12.131
host2=192.168.12.132
host3=192.168.12.133
src=/web/wwwroot/
dst1=web1
dst2=web2
dst3=web3
user1=web1user
user2=web3user
user3=web3user
```
* /usr/local/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib  $src \
* | while read files
        * do
        * /usr/bin/rsync -vzrtopg --delete --progress --password-file=/etc/server.pass $src $user1@$host1::$dst1
  * /usr/bin/rsync -vzrtopg --delete --progress --password-file=/etc/server.pass $src $user2@$host2::$dst2
  * /usr/bin/rsync -vzrtopg --delete --progress --password-file=/etc/server.pass $src $user3@$host3::$dst3
                * echo "${files} was rsynced" >>/tmp/rsync.log 2>&1
         * done
* 脚本相关解释如下：
* --timefmt：指定时间的输出格式。
* --format：指定变化文件的详细信息。
* 这两个参数一般配合使用，通过指定输出格式，输出类似与：
* 15/04/10 00:29 /web/wwwroot/ixdba.shDELETE,ISDIR was rsynced
* 15/04/10 00:30 /web/wwwroot/index.htmlMODIFY was rsynced
* 15/04/10 00:31 /web/wwwroot/pcre-8.02.tar.gzCREATE was rsynced
* 这个脚本的作用就是通过inotify监控文件目录的变化，进而触发rsync进行同步操作，由于这个过程是一种主动触发操作，通过系统内核完成的，所以，比起那些遍历整个目录的扫描方式，效率要高很多。
* 有时会遇到这样的情况：向inotify监控的目录（这里是/web/wwwroot/）写入一个很大文件时，由于写入这个大文件需要一段时间，此时inotify就会持续不停的输出该文件被更新的信息， 这样就会持续不停的触发rsync去执行同步操作，占用了大量系统资源，那么针对这种情况，最理想的做法是等待文件写完后再去触发rsync同步。 在这种情况下，可以修改inotify的监控事件，即：“-e close_write,delete,create,attrib”。
* 接着，将这个脚本命名为inotifyrsync.sh，放到/web/wwwroot目录下，然后给定可执行权限，放到后台运行：
* chmod 755 /web/wwwroot/inotifyrsync.sh
* /web/wwwroot/inotifyrsync.sh &
* 最后，将此脚本加入系统自启动文件：
* echo  “/web/wwwroot/inotifyrsync.sh &”>>/etc/rc.local
* 这样就完成了内容发布节点的所有配置工作。
* 4 测试rsync+inotify实时同步功能
 * 所有配置完成后，可以在网页发布节点的/web/wwwroot目录下添加、删除或者修改某个文件，然后到三个服务节点对应的目录查看文件是否跟随网页发布节点的/web/wwwroot目录下文件发生变化，如果你看到三个服务节点对应的目录文件跟着内容发布节点目录文件同步变化，那么我们这个业务系统就配置成功了。