---
title: 查找未使用的ip地址脚本
categories: linux
tag: hide
date: 2018-12-25 23:41:31
tags:
---
# 判断未使用的ip地址

```
ip=$1
#!/bin/bash
for i in `seq 1 255`;do
    ping  -w 1 -c 1 $ip$i >/dev/null
    if [ $? -eq 1 ]
    then
        echo $ip.$i not use
    fi
done

```