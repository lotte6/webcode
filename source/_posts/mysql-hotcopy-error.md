---
title: 运行mysqlhotcopy出现的错误修复
categories: tech
tag: hide
date: 2018-12-24 10:24:43
tags:
---
```
运行mysqlhotcopy出现的错误修复
错误一： 一看见Invalid db.table name类似的字句，就用下面的办法解决，这是mysql官网给出的办法。
错误：
[root@mail ~]# ./backup_mysqlhotcopy.sh 
Invalid db.table name 'mysql.mysql`.`columns_priv' at /usr/bin/mysqlhotcopy line 855.
Invalid db.table name 'policyd.policyd`.`blacklist' at /usr/bin/mysqlhotcopy line 855.
Invalid db.table name 'roundcubemail.roundcubemail`.`cache' at /usr/bin/mysqlhotcopy line 855.
Invalid db.table name 'vmail.vmail`.`admin' at /usr/bin/mysqlhotcopy line 855.
[root@test ~]# vi /usr/bin/mysqlhotcopy
用/dbh_tables查找找到
my @dbh_tables = eval { $dbh->tables() };
在它下面添加一行：
map { s/^.*?\.//o } @dbh_tables;
错误二：
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = (unset),
LC_ALL = (unset),
LANG = "en_CN.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
Invalid db.table name 'mysql.mysql`.`columns_priv' at 
/usr/bin/mysqlhotcopy line 855.
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = (unset),
LC_ALL = (unset),
LANG = "en_CN.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
Invalid db.table name 'policyd.policyd`.`blacklist' at 
/usr/bin/mysqlhotcopy line 855.
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = (unset),
LC_ALL = (unset),
LANG = "en_CN.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
Invalid db.table name 
'roundcubemail.roundcubemail`.`cache' at 
/usr/bin/mysqlhotcopy line 855.
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = (unset),
LC_ALL = (unset),
LANG = "en_CN.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
Invalid db.table name 'vmail.vmail`.`admin' at 
/usr/bin/mysqlhotcopy line 855.
具体解决方法是：
# echo $LANG (显示为"en_US.UTF-8:en_US:en_US.ISO-
8859-1")
en_US.UTF-8:en_US:en_US.ISO-8859-1
# echo $SUPPORTED (显示为"zh_HK.UTF-
8:zh_HK:zh:zh_CN.GB18030:zh_CN:zh:zh_TW.Big5:zh_TW:
zh:en_US.UTF-8:en_US:en:en_US.ISO-8859-1")
zh_HK.UTF-
8:zh_HK:zh:zh_CN.GB18030:zh_CN:zh:zh_TW.Big5:zh_TW:
zh:en_US.UTF-8:en_US:en:en_US.ISO-8859-1
# echo $LANGUAGE   (显示为空，我没有设定)
# echo $LC_ALL        (显示为空，没有设定)
# export LC_ALL=en_US
# export LANGUAGE=en_US
```