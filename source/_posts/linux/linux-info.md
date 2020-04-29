---
title: linux 信息查看
categories: linux
tag: hide
date: 2018-12-24 11:34:46
tags:
---
Linux查看机器配置方法命令
  cat /proc/cpuinfo

  或者vim /proc/cpuinfo

  >查看系统信息

  cat /proc/cpuinfo - CPU (i.e. vendor, Mhz, flags like mmx)

  cat /proc/interrupts - 中断

  cat /proc/ioports - 设备IO端口

  cat /proc/meminfo - 内存信息(i.e. mem used, free, swap size)

  cat /proc/partitions - 所有设备的所有分区

  cat /proc/pci - PCI设备的信息

  cat /proc/swaps - 所有Swap分区的信息

  cat /proc/version - Linux的版本号 相当于 uname -r

  uname -a - 看系统内核等信息

  >查看linux系统版本方法：

  cat /etc/redhat-release

  cat /etc/issue

  cat /proc/version

  >查看磁盘空间大小：

  df -m

  cat /etc/issue 查看操作系统版本

  cat /etc/inittab 查看启动项

  cat /proc/cpuinfo 查看cpu信息

  >uname -a 系统版本

  df -h 查硬盘

  cat /etc/passwd 查看所有用户的列表

  cat /etc/group 查看用户组

  du -sh 查看当前文件夹大小

  >这里linux下使用dmidecode查看硬件信息

  dmidecode is a tool for dumping a computer''''s DMI (some say SMBIOS) table contents in a human-readable format. This table contains a description of the system''''s hardware components, as well as other useful pieces of information such as serial numbers and BIOS revision. Thanks to this table, you can retrieve this information without having to probe for the actual hardware. While this is a good point in terms of report speed and safeness, this also makes the presented information possibly unreliable.

  dmidecode可以全面的显示bios、cpu、内存等硬件信息。

  >查看主板的序列号

  dmidecode | grep "Serial Number"

  >显示物理内存块数

  dmidecode |grep -A16 "Memory Device$"

  >显示CPU信息

  dmidecode |grep -A42 "Processor"|more

  >另外：

  grep -An (A和n之间也可以有空格) 输出包含指定字符串的行及该行后续的n行
   
    dmidecode | grep "Vendor" 服务器品牌
    dmidecode | grep "Product Name" 查看型号
  
  /usr/sbin/dmidecode | grep "Serial Number"可以读出计算机的标示号，当然这只对正规品牌的机器有效，如DELL、HP之类，取出的值和机器上贴的值是对应的，而类似清华同方之流的兼容机，基本上读不出任何有意义的数据。