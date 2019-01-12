---
title: linux 修改时区
categories: tech
tag: hide
date: 2018-12-19 00:42:53
tags:
---
```
\cp  /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && date -R
\cp  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && date -R
通过命令修改
tzselect
```