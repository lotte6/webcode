---
title: python时间,日期,时间戳处理 
categories: tech
tag: hide
date: 2018-12-18 22:06:53
tags:
---

1.将字符串的时间转换为时间戳
    方法:
        a = "2013-10-10 23:40:00"
        将其转换为时间数组
        import time
        timeArray = time.strptime(a, "%Y-%m-%d %H:%M:%S")
	转换为时间戳:
	timeStamp = int(time.mktime(timeArray))
	timeStamp == 1381419600
2.字符串格式更改
	如a = "2013-10-10 23:40:00",想改为 a = "2013/10/10 23:40:00"
	方法:先转换为时间数组,然后转换为其他格式
	timeArray = time.strptime(a, "%Y-%m-%d %H:%M:%S")
	otherStyleTime = time.strftime("%Y/%m/%d %H:%M:%S", timeArray)


3.时间戳转换为指定格式日期:
	方法一:
		利用localtime()转换为时间数组,然后格式化为需要的格式,如
		timeStamp = 1381419600
		timeArray = time.localtime(timeStamp)
		otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)
		otherStyletime == "2013-10-10 23:40:00"

	方法二:
		import datetime
		timeStamp = 1381419600
		dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
		otherStyleTime = dateArray.strftime("%Y-%m-%d %H:%M:%S")
		otherStyletime == "2013-10-10 23:40:00"
		注意：使用此方法时必须先设置好时区，否则有时差

4.获取当前时间并转换为指定日期格式
	方法一:
		import time
		获得当前时间时间戳
		now = int(time.time())  ->这是时间戳
		转换为其他日期格式,如:"%Y-%m-%d %H:%M:%S"
		timeArray = time.localtime(timeStamp)
		otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)

	方法二:
		import datetime
		获得当前时间
		now = datetime.datetime.now()  ->这是时间数组格式
		转换为指定的格式:
		otherStyleTime = now.strftime("%Y-%m-%d %H:%M:%S")

5.获得三天前的时间
	方法:
		import time
		import datetime
		先获得时间数组格式的日期
		threeDayAgo = (datetime.datetime.now() - datetime.timedelta(days = 3))
		转换为时间戳:
			timeStamp = int(time.mktime(threeDayAgo.timetuple()))
		转换为其他字符串格式:
			otherStyleTime = threeDayAgo.strftime("%Y-%m-%d %H:%M:%S")
	注:timedelta()的参数有:days,hours,seconds,microseconds

6.给定时间戳,计算该时间的几天前时间:
	timeStamp = 1381419600
	先转换为datetime
	import datetime
	import time
	dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
	threeDayAgo = dateArray - datetime.timedelta(days = 3)
	参考5,可以转换为其他的任意格式了	
	
7. 给定日期字符串，直接转换为datetime对象
	dateStr = '2013-10-10 23:40:00'
	datetimeObj = datetime.datetime.strptime(dateStr, "%Y-%m-%d %H:%M:%S")

        注：将字符串日期转换为datetime后可以很高效的进行统计操作，因为转换为datetime后，
           可以通过datetime.timedelta()方法来前后移动时间，效率很高，而且可读性很强。

8.计算两个datetime之间的差距
       a = datetime.datetime(2014,12,4,1,59,59)
       b = datetime.datetime(2014,12,4,3,59,59)
       diffSeconds = (b-a).total_seconds()
       


Python 时间获取 要使用到python time模块 代码如下:import time print time.time()结果: 1472483797.276373 结果为浮点型的 时间...


1.将字符串的时间转换为时间戳     方法:        import time a = "2013-10-10 23:40:00" 将其转换为时间数组 timeArray = time....
Sky_qingSky_qing2015年08月04日 10:4810756
 
成为 GitChat 超级会员，价格低至 99 元

从小白到大牛，经历不平凡！ 全年 Chat 免费订 · 优质资源任意享 · 领域专家畅快聊 · 精品课程随心学
python 时间戳和格式化时间的转化

python 里面与时间有关的模块主要是 time 和 datetime 如果想获取系统当前时间戳：time.time()　，是一个float型的数据 获取系统当前的时间信息 : time.ctime...
huiseguiji1huiseguiji12014年12月06日 11:463137
Python时间，日期，时间戳之间转换

1.将字符串的时间转换为时间戳 方法: a = "2013-10-10 23:40:00" 将其转换为时间数组 importtime timeArray = time.strptime(a,...
yaya1943yaya19432016年11月21日 13:414172
python 基础 —— 获取当前时间的时间戳

In [1]: import timeIn [2]: time.time() Out[2]: 1494902382.2486684In [3]: int(time.time()) Out[3]: 14...
HeatDeathHeatDeath2017年05月16日 10:404356
Python时间，日期，时间戳之间转换

Python时间，日期，时间戳之间转换  存储，学习，共享。。。 1.将字符串的时间转换为时间戳     方法:         a = "2013-10-10 23:40:00"    ...
wulantianwulantian2015年11月30日 09:117455
python timestamp和datetime之间的转换

1. 字符串日期时间转换成时间戳 '2015-08-28 16:43:37.283' --> 1440751417.283 或者 '2015-08-28 16:43:37' --> 144...
xxm524xxm5242015年08月28日 17:2319485
python time, datetime, string, timestamp相互转换

#########################Python Time Conversion #########################----...
abcd1f2abcd1f22016-03-22 16:204070
python时间处理之datetime

-*- coding: utf-8 -*- #datetime类 #datetime是date与time的结合体，包括date与time的所有信息。 #它的构造函数如下： #datet...
wirelessqawirelessqa2012-09-12 23:2158595
Python的time（时间戳与时间字符串互相转化）

#设a为字符串 import time a = "2011-09-28 10:00:00"   #中间过程，一般都需要将字符串转化为时间数组 time.strptime(a,'%Y-%m-%d %H:...
longshenlmjlongshenlmj2013-10-30 14:3446693
python unix时间戳与正常时间转化

有时候业务需要，需要把正常的时间格式与unix时间戳格式进行转换。       在python中转化方式如下,直接利用time中的函数: #! /usr/bin/env python #codin...
chivalrouslichivalrousli2016-04-10 23:122993
python - 获取时间戳（10位和13位）

在python 开发web程序时，需要调用第三方的相关接口，在调用时，需要对请求进行签名。需要用到unix时间戳。 在python里，在网上介绍的很多方法，得到的时间戳是10位。而java里默认是...
xuezhangjun0121xuezhangjun01212017-09-25 11:59501
Python 获取时间戳

Python date time 时间戳
a542551042a5425510422015-09-16 17:477141
python的datetime和unix时间戳之间相互转换

python的datetime和unix时间戳之间相互转换 将python的datetime转换为unix时间戳 import time import datetime dtime = da...
u012422446u0124224462016-09-23 15:519246
django自定义模板过滤器时间戳实例（python）

都要自定义模板过滤器了，创建项目直接略过了！ 视图部分： 1.APP目录下建一个templatetags的文件夹（规定这样的名字），文件夹内建一个__init__.py（空白的你懂的）和一个自己事务的...
wenyuanhaiwenyuanhai2017-06-23 22:57504
python如何得到13位时间戳？

python用time.time()得到的不是13位的时间戳，要怎么才能得到13位的？ python获取当前时间的unix时间戳 Unix timestamp：是从1970年1月1...
junli_chenjunli_chen2015-11-27 10:343139
Django 使用 MySQL 存储时间中遇到的问题(在数据库中记录插入时间、更新时间、删除时间)

一、MySQL 的时间存储格式 首先，把 MySQL 的时间类型做一下解释。在 MySQL 中，表示时间值的DATE和时间类型为 DATETIME、DATE、TIMESTAMP、TIME和YEAR。...
End0o0End0o02014-08-20 16:096165
