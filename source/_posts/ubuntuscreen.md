---
title: ubuntuscreen
categories: tech
tag: hide
date: 2018-12-13 19:21:56
tags:
---


最近把家里的显示器搬过来了，在加也享受双显示器的生活。
平时在家使用Ubuntu桌面，我的显示器是1440*900的，ubuntu默认下只有800*600、1204*768两个选择，在网上搜罗一翻后把分辨率给解决了。
此方法不需要安装显卡驱动，环境Ubuntu12.04.
首先运行：

$ cvt 1440 900
1440x900 59.89 Hz (CVT 1.30MA) hsync: 55.93 kHz; pclk: 106.50 MHz
Modeline "1440x900_60.00"  106.50  1440 1528 1672 1904  900 903 909 934 -hsync +vsync
记录得到的值：”1440x900_60.00″ 106.50 1440 1528 1672 1904 900 903 909 934 -hsync +vsync

查看当前显示设备：

$ xrandr
Screen 0: minimum 320 x 200, current 2390 x 768, maximum 8192 x 8192
LVDS1 connected 1366x768+1024+0 (normal left inverted right x axis y axis) 310mm x 174mm
   1366x768       59.6*+
   1360x768       59.8     60.0
   1024x768       60.0
   800x600        60.3     56.2
   640x480        59.9
VGA1 connected 1024x768+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   1024x768       60.0*
   800x600        60.3     56.2
   848x480        60.0
   640x480        59.9
HDMI1 disconnected (normal left inverted right x axis y axis)
DP1 disconnected (normal left inverted right x axis y axis)
其中LVDS1是笔记本本身的显示器，VGA1为新增的显示器，我们需要对VGA1进行设置。

添加Mode并设置VGA1
```
$ xrandr --newmode "1440x900" 106.50 1440 1528 1672 1904  900 903 909 934 -hsync +vsync
$ xrandr --addmode VGA1 1440x900
$ xrandr --output VGA1 --off
#--off 先关闭显示，避免桌面错乱
$ xrandr --output VGA1 --mode 1440x900 --auto --left-of LVDS1
#--left-of LVDS1 设置显示器顺序为在LVDS1的左侧，即窗口可向左拖动
```

如果需要添加多个分辨率，只需要执行上面的 cvt X*Y,得到值然后 xrandr –newmode –addmode
然后到Ubuntu的显示设置里面，可以看到有添加的分辨率可以调节了。

好吧，现在感觉不错了，可惜重启之后这些设置都失效了，怎么办？
好办，写个可执行脚本，开机自动运行。
```
#!/bin/bash
xrandr --newmode "1440x900" 106.50 1440 1528 1672 1904  900 903 909 934 -hsync +vsync
xrandr --addmode VGA1 1440x900
xrandr --output VGA1 --off
xrandr --output VGA1 --mode 1440x900 --auto --left-of LVDS1
```
保存为monitors.sh, sudo chmod +x monitors.sh
点右上角设置按钮–>启动应用程序–>添加

更多xrandr命令，xrandr –help

参考地址：  
https://wiki.ubuntu.com/X/Config/Resolution  
http://blog.logan.tw/2009/09/xrandr-vga.html  
http://ubuntuforums.org/showthread.php?t=1965529  