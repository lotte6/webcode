---
title: lvs做web应用
categories: tech
tag: hide
date: 2018-12-24 10:23:36
tags:
---
```
ifconfig eth0:0 192.168.0.9
ifconfig eth0:1 192.168.0.10
route add -host 192.168.0.9 dev eth0:0
echo "1">/proc/sys/net/ipv4/ip_forward
ipvsadm -C
ipvsadm -A -t 192.168.0.9:80 -s rr -p 120
ipvsadm -a -t 192.168.0.9:80 -r 192.168.0.10:80 -g
ipvsadm

ifconfig lo:0 192.168.0.9
route add -host 192.168.0.9 dev lo:0
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce 
```