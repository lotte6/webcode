---
title: shell_terminal_kuaijiejian
categories: linux
tag: hide
date: 2018-12-18 23:09:09
tags:
---

Linux终端中使用上一命令减少键盘输入
 
减少键盘输入，可以大大提高程序员的工作效率，快捷键的使用就是一个很好的例子。程序员经常使用终端。那么在终端上有没有类似的“快捷键”可以提高我们的效率呢？程序员的工作往往是前后相关连的。所以，本文将演示如何使用上一条命令提高工作效率的。
 
1.使用上一条命令的所有参数
 
方法：!*
 
例子：如果我对hello.txt和bye.txt进行了编辑，然后希望使用git add添加这两个文件。就可以使用：git add !*
 

 
2.使用上一条命令的最后一个参数
 
方法：!$
 
           ALT + .
 
           ESC + .
 
其中后面两种方法，terminal中会自动补全
 

 
3.使用上一条命令中除了最后一个参数的部分
 
方法：!-:
 
例子：个人认为这个比较有用，因为有些命令中间会输入一大堆选项，最后一个才是实际发挥作用的对象，如果再次输入选项，会显得麻烦。
 

 
4.使用上一条命令中任意一个部份
 
方法：ALT + <num> + .
 
            其中num表示的上一条命令中的第几部分，从0开始，对于ls -shld hello.txt。ALT +０＋. 就是ls。1就是-shld
 
5.替换上一条命令中的一个部份
 
方法：将foo替换为bar
 
            ^foo^bar 　　　仅替换地一个
 
　　　!!:gs/foo/bar   　替换所有
 

 
6.上一条命令
 
方法：！！
 
# 统计文件个数  
ls -alR  | grep -v '^-r' | grep -v '^d'

linux查看占用的CPU内存资源最多
 
linux下获取占用CPU资源最多的10个进程，可以使用如下命令组合： 
 
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head
 
linux下获取占用内存资源最多的10个进程，可以使用如下命令组合： 
 
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +4|head
 
命令组合解析（针对CPU的，MEN也同样道理）： 
 
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head
 
该命令组合实际上是下面两句命令： 
 
ps aux|head -1 ps aux|grep -v PID|sort -rn -k +3|head

## 统计内存使用
ps -A --sort -rss -o comm,pmem,pcpu |uniq -c |head -15
## 查找当前目录下文件
```
find ./* -prune
find . -maxdepth 1
```