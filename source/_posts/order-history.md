---
title: history 操作日志服务器
categories: tech
tag: hide
date: 2018-12-18 20:53:09
tags:
---


在/etc/profile加入以下内容
HISTFILESIZE=2000 
HISTSIZE=2000 
HISTTIMEFORMAT="%Y%m%d-%H%M%S: " 
export HISTTIMEFORMAT 
export PROMPT_COMMAND='{ command=$(history 1 | { read x y; echo $y; }); logger -p local1.notice -t bash -i "user=$USER,ppid=$PPID,from=$SSH_CLIENT,pwd=$PWD,command:$command"; }'

source /etc/profile

```
# vi /etc/rsyslog.conf

#增加如下行,IP自己换,也可以用域名,@表示用UDP协议,@@表示用TCP协议

*.*  @192.168.0.1
/etc/init.d/rsyslog restart

echo '*.*  @119.9.92.245' >>/etc/rsyslog.conf && /etc/init.d/rsyslog restart
```


服务端配置

首先修改log master机器上的/etc/rsyslog.conf文件，将其中下面四行的注释取消

$ModLoad imudp
$UDPServerRun 514
$ModLoad imtcp
$InputTCPServerRun 514

然后重新启动rsyslogd服务

/etc/init.d/rsyslog restart