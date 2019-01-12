---
title: mysql_replication
categories: tech
tag: hide
date: 2018-12-19 00:58:11
tags:
---

```
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1;

mysqldump  -uroot --default-character-set=latin1 novel pa_article >pa_article.sql
mysql  -uroot --default-character-set=latin1 novel <pa_article.sql

sudo mysqlhotcopy  -u root  novel  /mnt/ >log 2>&1

sudo mysql -e "stop slave;CHANGE MASTER TO
MASTER_HOST='121.18.211.48',
MASTER_USER='repl',
MASTER_PASSWORD='iamslave',
MASTER_LOG_FILE="mysql-bin.000014",
MASTER_LOG_POS=410336673"

sudo mysql -e "show slave status\G"

CHANGE MASTER TO
MASTER_HOST='121.18.211.50',
MASTER_USER='repl',
MASTER_PASSWORD='iamslave',
MASTER_LOG_FILE='mysql-bin.000404',
MASTER_LOG_POS=535567881;

grant select on *.* to 'read'@'121.18.211.%' identified by 'readnovel';
grant select on *.* to 'read'@'111.1.33.%' identified by 'readnovel';
grant all on *.* to 'write'@'111.1.33.%' identified by 'readnovel';
grant select on *.* to 'read'@'111.1.44.%' identified by 'readnovel';
grant all on *.* to 'write'@'111.1.44.%' identified by 'readnovel';
grant all on *.* to 'write'@'121.18.211.%' identified by 'readnovel';
grant select on *.* to 'read'@'58.83.146.%' identified by 'readnovel';
grant all on *.* to 'write'@'158.83.146.%' identified by 'readnovel';

mysql -e "show slave status\G" > status
mysqldump  -uroot --default-character-set=latin1 novel pa_article >pa_article.sql
mysqldump  -uroot --default-character-set=latin1 author pt_article >pt_article.sql
mysqlhotcopy  -u root  novel  /mnt/ >log 2>&1
mysqlhotcopy  -u root  author  /mnt/ >>log 2>&1
mysqlhotcopy  -u root  ucenter /mnt/ >>log 2>&1
mysqlhotcopy  -u root  uchome /mnt/ >>log 2>&1
mysqlhotcopy  -u root  pay /mnt/ >>log 2>&1
mysqlhotcopy  -u root  newuc /mnt/ >>log 2>&1
mysqlhotcopy  -u root  viewscount /mnt/ >>log 2>&1

stat
CHANGE MASTER TO
MASTER_HOST='121.18.211.44',
MASTER_USER='repl',
MASTER_PASSWORD='iamslave',
MASTER_LOG_FILE='mysql-bin.003725',
MASTER_LOG_POS=474875058;
Last_SQL_Error: Error 'The table 'beiker_user' is full' on query. Default database: 'beiker'. Query: 'insert into beiker_user (email,mobile,email_isavalible,mobile_isavalible,password,isavalible,customerkey,createdate,user_ip) values ('amberi_zhang@126.com','','0','0','HM2G9WrQrBU7JXFeRtGmJw==','0','3hiX87N4b786S996so5EQ1974AdcIn58D0J95258FS2lc5vX8iHiSd0N41r6','2012-11-02 10:08:12','119.164.135.122')'
1 row in set (0.00 sec)
```