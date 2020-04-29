---
title: 将Grub输出重定向到串口
categories: linux
tag: hide
date: 2018-12-19 00:03:10
tags:
---

* Step 1:　将Grub输出重定向到串口
* 编辑grub的配置文件/boot/grub/menu.lst, 添加如下行
* serial --unit=0 --speed=9600 --word=8 --parity=no --stop=1 terminal --timeout=10 serial console
* grub引导过程中，会将输出的同时发送到终端屏幕和串口．grub引导过程中将在终端和连接到串口的超级终端上提示：
* Press any key to continue
* Press any key to continue
* 每秒钟提示一次，共１０次．可修改menu.lst文件terminal行中的--timeout=10改变提示次数, 在这一段时间内, 可以在终端的键盘, 或者连接到串口的超级终端中按任意键进入grub选择菜单. 如果10秒内没有在终端和连接串口的超级终端上按任意键, 则grub的选择菜单将出现在连接串口的超级终端上, 如果希望默认情况下, grub选择菜单出现在终端上, 则可修改menu.lst将serial console修改为console serial.
* Step 2. 将kernel输出信息输出到串口
* 修改menu.lst的kernal行，　在该行后添加
* console=ttyS0, 9600n8 console=tty0
* 则kernel会将输出信息同时输出到串口和终端. 我的menu.list中修改过的记录如下:
* title Debian GNU/Linux, kernel 2.6.8-2-386
* root (hd0,0)
* kernel /vmlinuz-2.6.8-2-386 root=/dev/mapper/rootvg-root ro console=ttyS0,9600n8 console=tty0
* initrd /initrd.img-2.6.8-2-386
* savedefault boot
　* 在上例中, 服务启动的信息会显示在终端上(tty0), 如果进入单用户模式, 也只会在终端(tty0)上提示输入root密码, 如果需要将服务启动的信息也输出到串口上, 可修改两个console参数的顺序, 既修改为
* console=tty0 console=ttyS0,9600n8
* Step 3. 确认系统存在／sbin/agetty
* Step 4. 允许从串口登陆
* 修改/etc/inittab文件，增加如下内容
* T0:23:respawn:/sbin/getty -L ttyS0 9600 vt100
* Step 5.允许root用户通过串口登陆
* 修改／etc/securetty,添加以下行
* ttyS0
* 大功告成，可以测试了．
