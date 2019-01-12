---
title: shell_monitor502
categories: tech
tag: hide
date: 2018-12-18 21:53:13
tags:
---

```auto#!/bin/bash
D=`date -d "1 hours ago" +%Y-%m-%d:%H:%M`
ips="192.168.1.3
192.168.1.4
192.168.1.5
192.168.1.63
192.168.1.144"

for i in $ips;do
        error=$(ssh $i "awk -v D=`date -d "1 hours ago" +%Y:%H` 'BEGIN{\$sum=0}{if(\$4 ~ /\$D/ && \$9 ~ /502/)sum++}END{print sum}' /data/nginxlogs/bbs.goapk.com.access.log")
        echo "
$D#############################################################
bbs $i 502 $error" >> /var/log/502.log
        if [ ! -z ${error} ];then
                curl -I  http://118.26.224.22/shan/alert.php?a=3\&b1=3\&b2=25\&c="bbs $i 502 $error"
                echo "The alarm has been sent" >>/var/log/502.log
        fi
doneauto```