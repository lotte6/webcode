---
title: linux_pptpd
categories: linux
tag: hide
date: 2019-01-23 12:10:48
tags:
---
1. 下载工具包

wget  http://poptop.sourceforge.net/yum/stable/rhel5/x86_64/pptpd-1.4.0-1.rhel5.x86_64.rpm
rpm -Uvh pptpd-1.4.0-1.rhel5.x86_64.rpm
编辑 vi /etc/pptpd.conf 文件，确定本地VPN服务器的IP地址和客户端登录后分配的IP地址范围。pptpd.conf是PPTP服务PPTPD运行时使用的配置文件，常用选项如下(#号后是选项说明)：
```
　　option /etc/ppp/options.pptpd #PPP组件将使用的配置文件
　　stimeout 10 #开始PPTP控制连接的超时时间，以秒计
　　debug #把所有debug信息记入系统日志/var/log/messages
　　localip 192.168.1.4 #服务器VPN虚拟接口将分配的IP地址,修改成你的服务器地址
　　remoteip 192.168.1.100-200 #客户端VPN连接成功后将分配的IP或IP段，如果是地址
范围可表示为192.168.1.200-234的形式
``` 
2. 编辑 /etc/ppp/options.pptpd 文件，它是PPP功能组件pppd将使用的配置文件，由于PPTP VPN的加密和验证都与PPP相关，所以PPTP的加密和验证选项都将在这个配置文件中进行配置。
```
name pptpd  (服务器名服务器名称必须和chap-secrets称一致)
refuse-pap (拒绝pap身份认证模式)
refuse-chap (拒绝chap身份认证模式)
refuse-mschap (拒绝mschap身份认证模式)
require-mschap-v2  (在端点进行连接握手时需要使用微软的 mschap-v2 进行自身验证。)
require-mppe-128 (MPPE 模块使用 128 位加密。)
ms-dns 218.85.157.99  (ppp 为 Windows 客户端提供 DNS 服务器 IP 地址，第一个 ms-dns 为 DNS Master，第二个为 DNS Slave。)
proxyarp (建立 ARP 代理键值。)
debug (开启调试模式，相关信息同样记录在 /var/logs/message 中。)
lock (锁定客户端 PTY 设备文件。)
nobsdcomp (禁用 BSD 压缩模式。)
novj 
novjccomp (禁用 Van Jacobson 压缩模式。)
nologfd (禁止将错误信息记录到标准错误输出设备(stderr)。
```
3. 编辑/etc/ppp/chap-secrets文件  
pptpd server 连接帐户控制文件位于：/etc/ppp/chap-secrets.   chap-secrets 文件是有格式的，文件中每一行为一个客户端帐户，而一行又分为 4 段（用空格或者 TAB 分开），这 4 段从左到邮分别为：用户名、服务器名称*、密码、客户端分配到的 IP 地址。
```
#vi /etc/ppp/chap-secrets  
user                                 pptpd                              password                   IP
(帐户名 )       (此处要与options.pptpd的name一致)      (密码)      (客户端分配到的IP)
“Testvpn1 “                 pptpd                1               “*”
“Testvpn2 “                 pptpd                1              “192.169.1.120”
--注“* “表示VPN客户机IP地址由PPTP服务在地址段中进行选择
```
4 .设置IP伪装转发  
　　只有设置了IP伪装转发，通过VPN连接上来的远程计算机才能互相ping通，实现像局域网那样的共享。用下面的命令进行设置
```
#echo 1 > /proc/sys/net/ipv4/ip_forward
```
　　可以将这条命令放到文件/etc/rc.d/rc.local里面，以实现每次开机时自动运行该命令。
 
5. 打开防火墙端口
```
    要让外部用户能连接PPTP VPN，还需要在防火墙中加入以下规则（也就是将Linux服务器的1723端口和47端口打开，并打开GRE协议）
　　#/sbin/iptables -A INPUT -p tcp --dport 1723 -j ACCEPT
　　#/sbin/iptables -A INPUT -p tcp --dport 47 -j ACCEPT
　　#/sbin/iptables -A INPUT -p gre -j ACCEPT
```
6. 启动服务: /usr/local/sbin/pptpd
```
查看 ps –ef |grep pptpd  或者 netstat –atln|grep 1723
关闭服务  kill PID(pptpd的PID)
```
7. 查看日志 tail /var/log/messages
```
but I couldn't find any suitable secret (password) for it to use to do so.
/etc/ppp/chap-secrets
1)没有加入VPN用户信息或用户格式错误  
2)加载库文件时报版本错误

Plugin/usr/lib/pptpd/pptpd-logwtmp.so is for pppd version 2.4.3, this is 

  解决方法：
     （1）编辑 /usr/local/pptpd/etc/pptpd.conf 将logwtmp注释掉
     （2）编辑源码文件
/usr/local/src/software/pptpd-1.3.4/plugins/patchlevel.h将define VERSION “2.4.3″修改为：define VERSION“2.4.4″后，重新编译。

/usr/lib/pptpd/pptpd-logwtmp.so: cannot open shared object file: No such file or directory
解决方案：如果/usr/lib/pptpd目录不存在则创建，之后将pptpd-logwtmp.so(pptpd-1.3.4(解压文件夹)/plugins目录下)拷贝到该目录：
 
May 29 15:56:25 108test pptpd[27452]: MGR: PPP binary /usr/local/pptpd/sbin/pppd not executable
原因：在/etc/pptpd.conf 中 ppp 项制定了错误的pppd路径解决方案：正确指定正确的pppd路径
 

客户端设置好后，拨通之后，发现不能上网，原来是多了一条路由
   Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.0.0.200      10.0.0.200       1
解决方法：右击pptp连接属性--网络---属性---高级-----在远程网络上使用默认网络
 的√去掉
``` 


设置iptables规则访问外网

首先配置nat表的翻译规则, 将目标IP为192.168.0.0/24的包转向eth0接口. 在iptables配置文件的nat表中添加如下规则:

-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE 或者
-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j SNAT --to-source 119.9.95.141
若配置文件中还没有nat表的配置, 添加如下规则:

*nat
-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
COMMIT
然后配置filter表的规则, 在合适的位置((比如在-A INPUT -j DROP这种规则的前面))添加如下内容:

-A FORWARD -s 192.168.0.0/24 -j ACCEPT
-A FORWARD -d 192.168.0.0/24 -j ACCEPT
最后运行iptables-restore命令导入以上配置文件即可. 如果使用以上规则后无法连接成功，则在合适的位置添加如下规则:

-A INPUT -p tcp --dport 1723 -m state --state NEW -j ACCEPT
上面的规则允许使用tcp协议建立到1723端口的连接。pptpd默认监听1723端口，可以使用如下命令查看:

sudo netstat -nap | grep pptpd