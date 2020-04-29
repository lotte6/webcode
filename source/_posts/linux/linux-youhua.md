---
title: centos 系统优化
categories: linux
tag: hide
date: 2019-01-23 12:05:01
tags:
---
1. 安装所需工具
```
yum install  -y lrzsz sysstat htop curl wget bzip2 unzip zip nmap tree lynx iptraf vim-enhanced ntp gcc gcc-c++ rsync mtr telnet screen iftop iotop gcc-c++ git
```
2. 配置时间同步 
```
\cp  /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && date -R
echo "/usr/sbin/ntpdate cn.pool.ntp.org" >> /etc/cron.weekly/ntpdate
chmod +x /etc/cron.weekly/ntpdate 
```
3. 内核优化
```
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax =8587908743168
#kernel.shmmax        一般建议使用物理内存的一半以4G内存为例：4096/2*1024*1024=2147483648
kernel.shmall =8587908743168
#kernel.shmall        一般建议使用物理内存的一半以4G内存为例：4096/2*1024*1024=2147483648 以上两项数值如果填写大于本身物理内存则会不生效。
超过本身内存启动php会报错
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65535
```
4. 防火墙
```
# 设置iptables
if [ -f /etc/sysconfig/iptables ]; then
    cp /etc/sysconfig/iptables /etc/sysconfig/iptables.`date +"%Y-%m-%d_%H-%M-%S"`
fi
echo -e "*filter\n"\
":INPUT DROP [0:0]\n"\
"#:INPUT ACCEPT [0:0]\n"\
":FORWARD ACCEPT [0:0]\n"\
":OUTPUT ACCEPT [0:0]\n"\
"-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT\n"\
"-A INPUT -p icmp -m icmp --icmp-type any -j ACCEPT\n"\
"# setting trust ethernet.\n"\
"-A INPUT -i eth0 -j ACCEPT\n"\
"-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT\n"\
"-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT\n"\
"-A INPUT -d 127.0.0.1 -j ACCEPT\n"\
"-A INPUT -j DROP\n"\
"-A FORWARD -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m limit --limit 1/sec -j ACCEPT \n"\
"-A FORWARD -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK RST -m limit --limit 1/sec -j ACCEPT \n"\
"#-A FORWARD -p icmp -m icmp --icmp-type 8 -m limit --limit 1/sec -j ACCEPT \n"\
"COMMIT\n" > /etc/sysconfig/iptables
${SERVICE} iptables restart
```
5. 文件描述符
```
vim /etc/security/limits.conf 

*        soft   nproc  655350
*        hard   nproc  655350
*        soft   nofile  655350
*        hard   nofile  655350
vi /etc/security/limits.d/90-nproc.conf
*          soft    nproc    655350

cp /etc/security/limits.conf /etc/security/limits.conf.`date +"%Y-%m-%d_%H-%M-%S"`
sed -i '/# End of file/i\*\t\t-\tnofile\t\t65535' /etc/security/limits.conf 
```
6. 关闭不需要自启动，添加需要启动的
```
#!/bin/bash
SERVICES="acpid atd auditd avahi-daemon bluetooth cpuspeed cups firstboot hidd ip6tables isdn mcstrans messagebus pcscd rawdevices sendmail yum-updatesd"
for s in $SERVICES
do
    chkconfig $s off
    service $s stop
done 

chkconfig postfix off
chkconfig iptables off
chkconfig rpcbind off
chkconfig rpcgssd off
chkconfig rpcidmapd off
chkconfig nginx off
chkconfig mysqld off
chkconfig cups off
chkconfig fastcgi off
chkconfig vsftpd off
chkconfig portmap off
chkconfig portreserve off
chkconfig puppetmaster off
chkconfig avahi-daemon off
chkconfig ntpdate off
chkconfig ntpd off
chkconfig NetworkManager off
chkconfig bluetooth off
chkconfig certmonger off
chkconfig ip6tables off
chkconfig nfslock off

/etc/init.d/postfix stop
/etc/init.d/rpcbind stop
/etc/init.d/portreserve stop
/etc/init.d/avahi-daemon stop
/etc/init.d/ntpdate stop
/etc/init.d/ntpd stop


groupadd -g 1000 www
useradd -u 1000 -g 1000 www
mkdir /data/soft -p
mkdir /data/www -p
chown www.www /data/* -R
cat <<EOF > /etc/rsyncd.conf 
uid=root
gid=root
max connections=36000
use chroot=no
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/run/rsyncd.lock

[datasoft]
path=/data/soft
comment = m goapk att files
ignore errors = yes
read only = no
#hosts allow = 192.168.1.0/24 192.168.3.0/24 172.16.1.0/24
hosts deny = *
EOF

rsync --daemon
echo 'export HISTTIMEFORMAT="%y-%m-%d %H:%M:%S "'>> /etc/profile
```