---
title: mysql合集
categories: mysql
tag: hide
date: 2019-01-04 12:52:10
tags:
---
# 通过SQL语句查看MySQL数据库的表空间状态
```
SELECT

CONCAT(table_schema,’.’,table_name) AS ‘Table Name’,

table_rows AS ‘Number of Rows’,

CONCAT(ROUND(data_length/(1024*1024),6),’ MB’) AS ‘Data Size’,

CONCAT(ROUND(index_length/(1024*1024),6),’ MB’) AS ‘Index Size’,

CONCAT(ROUND((data_length+index_length)/(1024*1024),6),’ MB’) AS’Total Size’

FROM

information_schema.TABLES

WHERE

table_schema LIKE ‘database’;
```

# mysql审计
```
create database accesslog;
CREATE TABLE accesslog.accesslog (`id` int(11) primary key auto_increment, `time` timestamp, `localname` varchar(30), `matchname` varchar(30))
CREATE TABLE accesslog.accesslog (`id` int(11) , `time` timestamp, `localname` varchar(30), `matchname` varchar(30))；
grant select on accesslog.*  to  user@’%’;
在[mysqld]下添加以下设置：
init-connect='insert into      accesslog.accesslog(id,time,localname,matchname)values(connection_id(),now(),user(),current_user());'
```

# mysql binlog3种格式
```
mysql binlog3种格式，row,mixed,statement. 解析工作

mysqlbinlog --base64-output=DECODE-ROWS -v mysql-bin.000144 |more

--base64-output=DECODE-ROWS： 会显示出row模式带来的sql变更。
-v ：显示statement模式带来的sql语句
[mysql@002tmp]$ mysqlbinlog --base64-output=DECODE-ROWS -v mysql-bin.000144 |more
```

# handler error HA_ERR_KEY_NOT_FOUND handler error 
```
Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND handler error HA_ERR_KEY_NOT_FOUND
造成1032错误的根本原因是主从数据库数据不一致,导致同步操作在从库上无法执行.
造成1032错误的根本原因是主从数据库数据不一致,导致同步操作在从库上无法执行.
目前我所遇到的情况分为两种:
1 Replication 时使用了 主--binlog-ignore-db=db_name或者从--replicate-ignore-db=db_name.
假设 有两个库 pubs 和 test,忽略的是test,结果有这样一条sql 在 主上的test库执行:insert into pubs.tname values(XXXXX);
那么根据服务的配置,主上执行成功,从上没有执行,就会引发1032错误
2 TRIGGER 和 PROCEDURE的版本问题,如果在主从上版本不一致,例如主上的某个PROCEDURE执行后写入了5条数据,而从上执行后只写入了1行数据,这时,必然会引发1032错误
解决方法:
1 不使用 --binlog-ignore-db 和 --replicate-ignore-db=db_name
改为 从上 --replicate-wild-ignore-table=db_name.%
2 保证 主从 TRIGGER 和 PROCEDURE的版本一致

另一种方法，在主库导数据时带上master-date参数
master-date参数在建立slave数据库的时候会用到，当这个参数的值为1（默认情况下），mysqldump出来的文件就会包括CHANGE MASTER TO这个语句，CHANGE MASTER TO后面紧接着就是file和position的记录，file和position记录的位置就是slave从master端复制文件的起始位置。当这个值是2的时候，chang master to也是会写到dump文件里面去的，但是不会有上面那个作用了（thus is information only）
```

# mariadb报错
```
[ERROR] Can't find messagefile '/usr/share/mysql/english/errmsg.sys/errmsg.sys'

[mysqld]
pid-file        = /home/www/mysql/m3306/mysql3306.pid
log-error       = /home/www/mysql/m3306/mysql3306.err
language        = /usr/local/mysql3306/share/english   
```

# 字符集
```
character-set-server = utf8mb4
show char set like '%utf%';
```

# 编译
```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EXTRA_CHARSETS:STRING=utf8,gbk \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_DATADIR=/var/mysql/data \
-DWITH_EMBEDDED_SERVER=1  \
-DEXTRA_CHARSETS=all \
-DMYSQL_USER=mysql


DCMAKE_INSTALL_PREFIX=/usr/local/mysql #mysql安装的主目录，默认为/usr/local/mysql
DMYSQL_DATADIR=/usr/local/mysql/data #mysql数据库文件的存放目录，可以自定义
DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock #系统Socket文件（.sock）设置,基于该文件路径进行Socket链接，必须为绝对路径
DSYSCONFDIR=/etc #mysql配置文件 my.cnf的存放地址，默认为/etc下
DMYSQL_TCP_PORT=3306 #数据库服务器监听端口，默认为3306
DENABLED_LOCAL_INFILE=1 #允许从本地导入数据
DWITH_READLINE=1 #快捷键功能
DWITH_SSL=yes #支持 SSL
DMYSQL_USER=mysql #默认为mysql

//下面3个是数据库编码设置
DEXTRA_CHARSETS=all #安装所有扩展字符集，默认为all
DDEFAULT_CHARSET=utf8 #使用 utf8 字符
DDEFAULT_COLLATION=utf8_general_ci #校验字符

//下面5个是数据库存储引擎设在
DWITH_MYISAM_STORAGE_ENGINE=1 #安装 myisam 存储引擎
DWITH_INNOBASE_STORAGE_ENGINE=1 #安装 innodb 存储引擎
DWITH_ARCHIVE_STORAGE_ENGINE=1 #安装 archive 存储引擎
DWITH_BLACKHOLE_STORAGE_ENGINE=1 #安装 blackhole 存储引擎
DWITH_PARTITION_STORAGE_ENGINE=1 #安装数据库分区
```

# mysql自动分区脚本
```
#!/bin/sh
#
#
:<<BLOCK
######################################################################
SHELL_NAME：Logdb_Add_Partition.sh
  Functional Description：At the last month auto add the logdb table partition
    Argument：

            $1 USER Mysql Account
            $2 PASS Mysql Account Pass
            $3 DB        Mysql Logdb
    Version：V1.0
    Creater : SongYunkui
              Colin_Song
    Crete_time：2010/12/9
    Modify：1. MODIFY BY 
            1. ADD BY ____ ___/__/__ Add:_________
######################################################################            
BLOCK

#######################################################################################
if [ $# -lt 3 ]; then
  echo "Please Input The Correct Args"
  echo "Usage Logdb_Add_Partition.sh <USER> <PASS> <DB>"
  exit -1
fi

USER=$1
PASS=$2
DB=$3

##config section begin
CONN_MYSQL="-u$USER -p$PASS -s"
MYSQL_HOME=/opt/modules/mysql
MYSQL_DIR=${MYSQL_HOME}/bin/mysql
SHELL_BASE=/opt/sbin/Logdb
LOG_DIR=${SHELL_BASE}/log
OPT_NAME=add_partition

MKDIR=`whereis -b mkdir|awk '{print $2}'`
TOUCH=`whereis -b touch|awk '{print $2}'`
DATE=`whereis -b date|awk '{print $2}'`
if [ ! -d ${SHELL_BASE} ]
then
    ${MKDIR} -p ${SHELL_BASE}
fi

if [ ! -d ${LOG_DIR} ]
then
    ${MKDIR} -p ${LOG_DIR}
fi

if [ ! -d ${INI_DIR} ]
then
    ${MKDIR} -p ${INI_DIR}
fi

LOG_FILE=${LOG_DIR}/${OPT_NAME}.log

#config section end

#working start
CURRENT_DATE=`${DATE} +'%Y-%m-%d'`
echo "${CURRENT_DATE} everything is ok, runing start" >> ${LOG_FILE}

#loop read the partition table and column
while read TAB_NAME COL_NAME
do

COUNTER=1
CURRENT_YEAR=`date +%Y`
#check the next month
NEXT_MONTH=`date -d next-month +%m`

#check the next month has many days
case ${NEXT_MONTH} in
            1|01|3|03|5|05|7|07|8|08|10|12)
                                           CURRENT_DAY=31
            ;;
            4|04|6|06|9|09|11)
                              CURRENT_DAY=30
            ;;
            2|02)
                if [ `expr ${CURRENT_YEAR} % 4` -eq 0 ]; then
                        if [ `expr ${CURRENT_YEAR} % 400` -eq 0 ]; then
                                CURRENT_DAY=29
                        elif [ `expr ${CURRENT_YEAR} % 100` -eq 0 ]; then
                                CURRENT_DAY=28
                        else
                                CURRENT_DAY=29
                        fi
                else
                        CURRENT_DAY=28
                fi
             ;;
esac

#work start add the every day partition
while [ ${COUNTER} -le ${CURRENT_DAY} ]

do

#calculate the current day's next {counter} day
PATNAME_DATE=`date -d "${COUNTER} days" +%Y%m%d`
COUNTER=`expr ${COUNTER} + 1`
PAT_DATE=`date -d "${COUNTER} days" +%Y%m%d`

#change the unix_timestamp
PAT_UNIX_TIMESTAMP=`${MYSQL_DIR} ${CONN_MYSQL} <<EOF
use ${DB};
select UNIX_TIMESTAMP('${PAT_DATE}'); 
EOF`

##add partition sql
V_SQL="ALTER TABLE ${DB}."${TAB_NAME}" ADD PARTITION (PARTITION P"${PATNAME_DATE}" VALUES LESS THAN ("${PAT_UNIX_TIMESTAMP}"));"
echo $V_SQL

#exec the sql
${MYSQL_DIR} ${CONN_MYSQL} <<EOF
use ${DB};
$V_SQL; 
EOF

done
done<./Logdb_Partition.ini

#working end
END_DATE=`${DATE} +'%Y-%m-%d %H:%M:%S'`
echo "${END_DATE} runing finished" >> ${LOG_FILE}
echo -e "\n--------------------------------------------------------------------" >> ${LOG_FILE}
```

# mysql 常用操作
```
Drop table操作自动回收表空间，如果对于统计分析或是日值表，删除大量数据后可以通过:alter table TableName engine=innodb;回缩不用的空间。
innodb_file_per_table

show variables like '%per_table%';

FLUSH TABLES WITH READ LOCK 所有数据库只读
lock table 123 read
unlock tables 解锁
show OPEN TABLES where In_use > 0; 查看被锁表
show function status;
SELECT * from information_schema.VIEWS\G
show procedure status;
show create procedure proc_name;
show create function func_name;

mysqldump -uroot -p -n -t -d -R --triggers=false 数据库名 > 文件名


#导出某个数据库－－结构+数据
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt db_name |gzip -9 > /db_bakup/db_name.gz

#导出某个数据库的表－－结构+数据+函数+存储过程
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt -R db_name |gzip -9 > /db_backup/db_name.gz

#导出多个数据库
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --databases db_name1 db_name2 db_name3 |gzip -9 > /db_backup/mul_db.gz 

#导出所有的数据库
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --all-databases |gzip -9 > /db_bak/all_db.gz

#导出某个数据库的结构
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --no-data db_name|gzip -9 > /db_bak/db_name.strcut.gz

#导出某个数据库的数据
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --no-create-info db_name|gzip -9 > /db_bak/db_naem.data.gz

#导出某个数据库的某张表
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt db_name tbl_name |gzip -9 > /db_bak/db_name.tal_name.gz

# 导出某个数据库的某张表的结构
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --no-data db_name tal_name | gzip -9 > /db_bak/db_name.tal_name.struct.gz

#导出某个数据库的某张表的数据
shell>mysqldump -h192.168.161.124 -uroot -pxxxxxx --opt --no-create-info db_name tbl_name | gzip -9 > /db_bak/db_name.tbl_name.data.gz

##--opt==--add-drop-table + --add-locks + --create-options + --disables-keys + --extended-insert + --lock-tables + --quick + --set+charset
##默认使用--opt，--skip-opt禁用--opt参数

##删除主键及自增长
ALTER TABLE u_open_platform_user_manage DROP PRIMARY KEY,change id id bigint(20);
```

# Mysqldump
```
Mysqldump参数大全（参数来源于mysql5.5.19源码）

 

参数

参数说明

--all-databases  , -A

导出全部数据库。

mysqldump  -uroot -p --all-databases

--all-tablespaces  , -Y

导出全部表空间。

mysqldump  -uroot -p --all-databases --all-tablespaces

--no-tablespaces  , -y

不导出任何表空间信息。

mysqldump  -uroot -p --all-databases --no-tablespaces

--add-drop-database

每个数据库创建之前添加drop数据库语句。

mysqldump  -uroot -p --all-databases --add-drop-database

--add-drop-table

每个数据表创建之前添加drop数据表语句。(默认为打开状态，使用--skip-add-drop-table取消选项)

mysqldump  -uroot -p --all-databases  (默认添加drop语句)

mysqldump  -uroot -p --all-databases –skip-add-drop-table  (取消drop语句)

--add-locks

在每个表导出之前增加LOCK TABLES并且之后UNLOCK  TABLE。(默认为打开状态，使用--skip-add-locks取消选项)

mysqldump  -uroot -p --all-databases  (默认添加LOCK语句)

mysqldump  -uroot -p --all-databases –skip-add-locks   (取消LOCK语句)

--allow-keywords

允许创建是关键词的列名字。这由表名前缀于每个列名做到。

mysqldump  -uroot -p --all-databases --allow-keywords

--apply-slave-statements

在'CHANGE MASTER'前添加'STOP SLAVE'，并且在导出的最后添加'START SLAVE'。

mysqldump  -uroot -p --all-databases --apply-slave-statements

--character-sets-dir

字符集文件的目录

mysqldump  -uroot -p --all-databases  --character-sets-dir=/usr/local/mysql/share/mysql/charsets

--comments

附加注释信息。默认为打开，可以用--skip-comments取消

mysqldump  -uroot -p --all-databases  (默认记录注释)

mysqldump  -uroot -p --all-databases --skip-comments   (取消注释)

--compatible

导出的数据将和其它数据库或旧版本的MySQL 相兼容。值可以为ansi、mysql323、mysql40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_tables_options、no_field_options等，

要使用几个值，用逗号将它们隔开。它并不保证能完全兼容，而是尽量兼容。

mysqldump  -uroot -p --all-databases --compatible=ansi

--compact

导出更少的输出信息(用于调试)。去掉注释和头尾等结构。可以使用选项：--skip-add-drop-table  --skip-add-locks --skip-comments --skip-disable-keys

mysqldump  -uroot -p --all-databases --compact

--complete-insert,  -c

使用完整的insert语句(包含列名称)。这么做能提高插入效率，但是可能会受到max_allowed_packet参数的影响而导致插入失败。

mysqldump  -uroot -p --all-databases --complete-insert

--compress, -C

在客户端和服务器之间启用压缩传递所有信息

mysqldump  -uroot -p --all-databases --compress

--create-options,  -a

在CREATE TABLE语句中包括所有MySQL特性选项。(默认为打开状态)

mysqldump  -uroot -p --all-databases

--databases,  -B

导出几个数据库。参数后面所有名字参量都被看作数据库名。

mysqldump  -uroot -p --databases test mysql

--debug

输出debug信息，用于调试。默认值为：d:t:o,/tmp/mysqldump.trace

mysqldump  -uroot -p --all-databases --debug

mysqldump  -uroot -p --all-databases --debug=” d:t:o,/tmp/debug.trace”

--debug-check

检查内存和打开文件使用说明并退出。

mysqldump  -uroot -p --all-databases --debug-check

--debug-info

输出调试信息并退出

mysqldump  -uroot -p --all-databases --debug-info

--default-character-set

设置默认字符集，默认值为utf8

mysqldump  -uroot -p --all-databases --default-character-set=latin1

--delayed-insert

采用延时插入方式（INSERT DELAYED）导出数据

mysqldump  -uroot -p --all-databases --delayed-insert

--delete-master-logs

master备份后删除日志. 这个参数将自动激活--master-data。

mysqldump  -uroot -p --all-databases --delete-master-logs

--disable-keys

对于每个表，用/*!40000 ALTER TABLE tbl_name DISABLE KEYS */;和/*!40000 ALTER TABLE tbl_name ENABLE KEYS */;语句引用INSERT语句。这样可以更快地导入dump出来的文件，因为它是在插入所有行后创建索引的。该选项只适合MyISAM表，默认为打开状态。

mysqldump  -uroot -p --all-databases 

--dump-slave

该选项将导致主的binlog位置和文件名追加到导出数据的文件中。设置为1时，将会以CHANGE MASTER命令输出到数据文件；设置为2时，在命令前增加说明信息。该选项将会打开--lock-all-tables，除非--single-transaction被指定。该选项会自动关闭--lock-tables选项。默认值为0。

mysqldump  -uroot -p --all-databases --dump-slave=1

mysqldump  -uroot -p --all-databases --dump-slave=2

--events, -E

导出事件。

mysqldump  -uroot -p --all-databases --events

--extended-insert,  -e

使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项。

mysqldump  -uroot -p --all-databases

mysqldump  -uroot -p --all-databases--skip-extended-insert   (取消选项)

--fields-terminated-by

导出文件中忽略给定字段。与--tab选项一起使用，不能用于--databases和--all-databases选项

mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-terminated-by=”#”

--fields-enclosed-by

输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项

mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-enclosed-by=”#”

--fields-optionally-enclosed-by

输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项

mysqldump  -uroot -p test test --tab=”/home/mysql”  --fields-enclosed-by=”#” --fields-optionally-enclosed-by  =”#”

--fields-escaped-by

输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项

mysqldump  -uroot -p mysql user --tab=”/home/mysql” --fields-escaped-by=”#”

--flush-logs

开始导出之前刷新日志。

请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs。

mysqldump  -uroot -p --all-databases --flush-logs

--flush-privileges

在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候。

mysqldump  -uroot -p --all-databases --flush-privileges

--force

在导出过程中忽略出现的SQL错误。

mysqldump  -uroot -p --all-databases --force

--help

显示帮助信息并退出。

mysqldump  --help

--hex-blob

使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用该选项。影响到的字段类型有BINARY、VARBINARY、BLOB。

mysqldump  -uroot -p --all-databases --hex-blob

--host, -h

需要导出的主机信息

mysqldump  -uroot -p --host=localhost --all-databases

--ignore-table

不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如：--ignore-table=database.table1 --ignore-table=database.table2 ……

mysqldump  -uroot -p --host=localhost --all-databases --ignore-table=mysql.user

--include-master-host-port

在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  

mysqldump  -uroot -p --host=localhost --all-databases --include-master-host-port

--insert-ignore

在插入行时使用INSERT IGNORE语句.

mysqldump  -uroot -p --host=localhost --all-databases --insert-ignore

--lines-terminated-by

输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。

mysqldump  -uroot -p --host=localhost test test --tab=”/tmp/mysql”  --lines-terminated-by=”##”

--lock-all-tables,  -x

提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项。

mysqldump  -uroot -p --host=localhost --all-databases --lock-all-tables

--lock-tables,  -l

开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表。

请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。

mysqldump  -uroot -p --host=localhost --all-databases --lock-tables

--log-error

附加警告和错误信息到给定文件

mysqldump  -uroot -p --host=localhost --all-databases  --log-error=/tmp/mysqldump_error_log.err

--master-data

该选项将binlog的位置和文件名追加到输出文件中。如果为1，将会输出CHANGE MASTER 命令；如果为2，输出的CHANGE  MASTER命令前添加注释信息。该选项将打开--lock-all-tables 选项，除非--single-transaction也被指定（在这种情况下，全局读锁在开始导出时获得很短的时间；其他内容参考下面的--single-transaction选项）。该选项自动关闭--lock-tables选项。

mysqldump  -uroot -p --host=localhost --all-databases --master-data=1;

mysqldump  -uroot -p --host=localhost --all-databases --master-data=2;

--max_allowed_packet

服务器发送和接受的最大包长度。

mysqldump  -uroot -p --host=localhost --all-databases --max_allowed_packet=10240

--net_buffer_length

TCP/IP和socket连接的缓存大小。

mysqldump  -uroot -p --host=localhost --all-databases --net_buffer_length=1024

--no-autocommit

使用autocommit/commit 语句包裹表。

mysqldump  -uroot -p --host=localhost --all-databases --no-autocommit

--no-create-db,  -n

只导出数据，而不添加CREATE DATABASE 语句。

mysqldump  -uroot -p --host=localhost --all-databases --no-create-db

--no-create-info,  -t

只导出数据，而不添加CREATE TABLE 语句。

mysqldump  -uroot -p --host=localhost --all-databases --no-create-info

--no-data, -d

不导出任何数据，只导出数据库表结构。

mysqldump  -uroot -p --host=localhost --all-databases --no-data

--no-set-names,  -N

等同于--skip-set-charset

mysqldump  -uroot -p --host=localhost --all-databases --no-set-names

--opt

等同于--add-drop-table,  --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys 该选项默认开启,  可以用--skip-opt禁用.

mysqldump  -uroot -p --host=localhost --all-databases --opt

--order-by-primary

如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间。 

mysqldump  -uroot -p --host=localhost --all-databases --order-by-primary

--password, -p

连接数据库密码

--pipe(windows系统可用)

使用命名管道连接mysql

mysqldump  -uroot -p --host=localhost --all-databases --pipe

--port, -P

连接数据库端口号

--protocol

使用的连接协议，包括：tcp, socket, pipe, memory.

mysqldump  -uroot -p --host=localhost --all-databases --protocol=tcp

--quick, -q

不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项。

mysqldump  -uroot -p --host=localhost --all-databases 

mysqldump  -uroot -p --host=localhost --all-databases --skip-quick

--quote-names,-Q

使用（`）引起表和列名。默认为打开状态，使用--skip-quote-names取消该选项。

mysqldump  -uroot -p --host=localhost --all-databases

mysqldump  -uroot -p --host=localhost --all-databases --skip-quote-names

--replace

使用REPLACE INTO 取代INSERT INTO.

mysqldump  -uroot -p --host=localhost --all-databases --replace

--result-file,  -r

直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用。

mysqldump  -uroot -p --host=localhost --all-databases --result-file=/tmp/mysqldump_result_file.txt

--routines, -R

导出存储过程以及自定义函数。

mysqldump  -uroot -p --host=localhost --all-databases --routines

--set-charset

添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。

mysqldump  -uroot -p --host=localhost --all-databases 

mysqldump  -uroot -p --host=localhost --all-databases --skip-set-charset

--single-transaction

该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK  TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。

mysqldump  -uroot -p --host=localhost --all-databases --single-transaction

--dump-date

将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项。

mysqldump  -uroot -p --host=localhost --all-databases

mysqldump  -uroot -p --host=localhost --all-databases --skip-dump-date

--skip-opt

禁用–opt选项.

mysqldump  -uroot -p --host=localhost --all-databases --skip-opt

--socket,-S

指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock

mysqldump  -uroot -p --host=localhost --all-databases --socket=/tmp/mysqld.sock

--tab,-T

为每个表在给定路径创建tab分割的文本文件。注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上。

mysqldump  -uroot -p --host=localhost test test --tab="/home/mysql"

--tables

覆盖--databases (-B)参数，指定需要导出的表名。

mysqldump  -uroot -p --host=localhost --databases test --tables test

--triggers

导出触发器。该选项默认启用，用--skip-triggers禁用它。

mysqldump  -uroot -p --host=localhost --all-databases --triggers

--tz-utc

在导出顶部设置时区TIME_ZONE='+00:00' ，以保证在不同时区导出的TIMESTAMP 数据或者数据被移动其他时区时的正确性。

mysqldump  -uroot -p --host=localhost --all-databases --tz-utc

--user, -u

指定连接的用户名。

--verbose, --v

输出多种平台信息。

--version, -V

输出mysqldump版本信息并退出

--where, -w

只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来。

mysqldump  -uroot -p --host=localhost --all-databases --where=” user=’root’”

--xml, -X

导出XML格式.

mysqldump  -uroot -p --host=localhost --all-databases --xml

--plugin_dir

客户端插件的目录，用于兼容不同的插件版本。

mysqldump  -uroot -p --host=localhost --all-databases --plugin_dir=”/usr/local/lib/plugin”

--default_auth

客户端插件默认使用权限。

mysqldump  -uroot -p --host=localhost --all-databases --default-auth=”/usr/local/lib/plugin/<PLUGIN>”
```