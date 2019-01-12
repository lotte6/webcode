---
title: linux_encrypt
categories: tech
tag: hide
date: 2018-12-19 00:22:09
tags:
---

```
[root@centos54 tmp]# /usr/local/src/shc-3.8.7/shc -e 20/10/2010 -m "lianxi aaa@163.com" -v -r -f ./ex.sh
-e:指定过期时间为2010年10月20日
-m:过期后打印出的信息；
-v: verbose
-r: 可在相同操作系统的不同主机上执行
-f: 指定源shell
 
方法:
shc -r -f script-name
注意:要有-r选项, -f 后跟要加密的脚本名.
运行后会生成两个文件,script-name.x 和 script-name.x.c
script-name.x是加密后的可执行的二进制文件.
./script-name.x 即可运行.
script-name.x.c是生成script-name.x的原文件(c语言)
 
说明：
经我测试，相同在操作系统，shc后的可执行二进制文件直接可以移植运行，但不同操作系统可能会出现问题，如我将源shell在CentOS5.4上加密后移到redhat as5u4上不能运行，出现“Floating point exception”错误提示，但移到另一台CentOS5.4上直接运行没问题。

问题处理：
#shc -r -f test.sh
# ./test.sh.x 
报错：
[3]+  Stopped                 ./test.sh.x

需要加个 -T参数：
shc -v -r -T -f test.sh
# ./test.sh.x
成功
shc帮助没有-T参数，但可以使用，也许是版本问题，不再深究。
```