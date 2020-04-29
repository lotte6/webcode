---
title: mysql_partition
categories: mysql
tag: hide
date: 2019-01-04 15:47:25
tags:
---
```
分区概念
分区针对不同的数据库，具有不同的特性。在这里专门针对MySQL数据库而言。在MySQL数据库里，分区这个概念是从mysql 5.1才开始提供的。不过目前只有在mysql advanced版本里才提供。

分区是把数据库、或它的组成部分(比如表)分成几个小部分。而且专门介绍的都是’水平分区’，即对表的行进行划分。


分区的优点
1. 可以提高数据库的性能；
2. 对大表(行较多)的维护更快、更容易，因为数据分布在不同的逻辑文件上；
3. 删除分区或它的数据是容易的，因为它不影响其他表。
    
注意：pruning，即截断。意思是说当你查询时，只扫描所需要查询的分区。。其他部分不会扫描。。这就大大地提高了性能。


分区类型
分区具有如下4种类型：
Range分区：是对一个连续性的行值，按范围进行分区；比如：id小于100；id大于100小于200；
List分区：跟range分区类似，不过它存放的是一个离散值的集合。
Hash分区：对用户定义的表达式所返回的值来进行分区。可以写partitions (分区数目)，或直接使用分区语句,比如partition p0 values in…..。
Key分区：与hash分区类似，只不过分区支持一列或多列，并且MySQL服务器自身提供hash函数。

具体描述：
分区语法：
create table t(id int,name varchar(20)) engine=myisam partition by range(id);

按range范围进行分区：
create table orders_range
(
id int auto_increment primary key,
customer_surname varchar (30),
store_id int,
salesperson_id int,
order_Date date,
note varchar(500)
)  engine=myisam 
partition by range(id) 
(
partition p0 values less than(5),
partition p1 values less than(10),
partition p3 values less than(15)
);
其实上面的分区创建，我们可知道，它的表类型为myisam，而每个分区的引擎也是myisam，这个可以通过show create table tablename查看。当我们插入数据到表里时，如果要查看小于8的信息，它之后检索p0和p12个分区。这样就非常快速了。

按list进行分区：
create table orders_list
(
id int auto_increment,
customer_surname varchar(30),
store_id int,
salesperson_id int,
order_Date date,
note varchar(500),
index idx(id)
) engine=myisam  partition by list(store_id) 
(
partition p0 values in(1,3),
partition p1 values in2,4,6),
partition p3 values in(10)
);

list 分区只能把你插入的值放在某个已定的分区里，若没有那个值，，就显示不能插入。

按hash进行分区：
create table orders_hash
(
id int auto_increment primary key,
cutomer_surname varchar(30),
store_id int,
salesperon_id int,
order_date date,
note varcahr(500)
) engine=myisam  partition by hash(id) partitions 4;

如果分为4个分区，那当我插入数据时，哪些数据是放在哪些分区里呢？ 当我对某个id值进行检索时，它明确说放到哪个分区里？或者说是有什么内部机制？
使用hash分区，最主要就是确保数据的分配，它是基于create table时提供的表达式。不必定义单独的分区，只要使用partitions关键字和所需要分多少个区的数字。语句如上所述。

按key进行分区：
create table orders_key
(
id int auto_increment,
customer_surname varchar(30),
store_id int,
alesperson_id int,
order_Date date,
note varcahr(500),
index_idx(id)
) engine=myisam  partition by key(order_date) partitions 4;
这个分区类似于hash分区，除了MySQL服务器使用它本身的hash表达式，不像其他类型的分区，不必要求使用一个int或null的表达式。

按子分区进行分区：
create table orders_range
(
id int auto_increment primary key,
customer_surname varchar(30),
store_id int,
salesperson_id int,
order_Date date,
note varchar(500)
) engine=myisam  partition by range(id) 
subpartition by hash(store_id) subpartitions 2
(
partition p0 values less than(5),
partition p1 values less than(10),
partition p3 values less than(15)
);
当把数据插入到表中时，那什么数据是放在子分区里呢？

================================================
MySQL partition分区II（ 续）
获得分区信息
MySQL可以通过如下方式来获取分区表的信息:
Show create tabe table;      //表详细结构
show table status;     //表的各种参数状态
select * from information_schema.partitions；//通过数据字典来查看表的分区信息
explain partitions select * from table;   // 通过此语句来显示扫描哪些分区，及他们是如何使用的.


对分区进行修改 (修改、合并、重定义分区)
修改分区
修改部分分区：
由于我们平常使用的数据库大都是动态运行的，所以只对某个表分区进行修改就OK了。
可以对range或list表分区进行add或drop，也可以对hash或key分区表进行合并或分解。这些动作都在alter table语句里进行。
使用add partition 关键字来对已有分区表进行添加。
Alter 
table 
orders_range
add 
partition 
(
Partition p5 values less than(maxvalue)
)

Reorganize partition关键字可以对表的部分分区或全部分区进行修改，并且不会丢失数据。
Splitting即分解一个已有分区：
Alter table orders_range
reorganize partition p0 into

(
partition n0 values less than(5000),
partition n1 values less than(10000)
);

Merge分区：像上面把p0分成n0和n1，现在在把2个合并为一个。
Alter table orders_range reorganize partition n0,n1 into
(
Partition p0 values less than(10000)
);

修改所有的分区：在into关键字之前或之后都指定多个分区
Alter table orders_range reorganize partition p0,p1,p2,p3,p4,p5 into
(
Partition r0 values less than(25000),
Partition r1 values less than(50000),
Partition r2 values less than(maxvalue)
);

Coalesce 合并分区：
Merge分区的另一种方法就是alter table….coalesce partition语句，你不能对hash或key分区进行删除
Alter table orders_key coalesce partition1;

Redefine重定义分区
Alter table orders_range partition by hash(id) partitions 4;

对分区进行删除 (删除、删除所有分区)
Drop 分区：
可以对range或list类型的分区通过drop partition 关键字进行删除
Alter table orders_range drop partition p0;

注意：
1.对这个分区进行删除时，你会把这个分区的所有数据进行删除，与delete语句相等；
2.在做alter table..drop partition时，必须有drop权限；
3.运行这个删除命令，它不会返回删除了的行，可以通过select count()语句查看。
如果想对多个分区进行删除，可以使用如下命令语句：Alter table orders_range drop partition p1,p2;


删除所有分区
通过如下命令语句删除表中所有分区，最后是一个正规表.
Alter table orders_range remove partitioning;



当进行分区操作，了解对性能所产生的影响是非常有帮助的：
1.创建分区表比无分区的正规表要稍微慢些；
2.通过alter table….drop partition语句进行删除比delete语句要快些；
3.在range或list分区类型上添加分区(alter table…add partition语句)是相当快的，因为没有移动数据到新分区里。
4.当在一个key或hash类型的分区上执行alter table….add partition语句，要依赖表中已有多少行记录，数据越多，它添加一个新分区的时间就越长。当创建一个表时，使用线性hash或线性key分区是相当快的。
5.对成百上千的行记录，进行alter table …coalesce partition, alter table …reorganize partition, alter table…partition by操作命令时，是相当慢的。
6.当使用add partition命令时，线性hash和线性key分区会使coalesce partition操作更快， alter table …remove partitioning比其他都要快，因为mysql没有要求哪个文件来替代行，即使是移动数据。


各种存储引擎的分区
MySQL分区可以对所有MySQL支持的存储引擎进行分区，比如：myisam, innodb, archive, NDBcluster(只可以线性key),falcon， 不支持分区的引擎：merge, federated, csv, blackhole

注意：所有分区和子分区的表类型要一致；
      索引维护要依赖表类型；
      锁住某些行，也依赖于存储引擎；
      分区也属于存储引擎的顶层，所以进行update和insert时，性能不会产生很大的影响。

各种存储引擎使用分区时的限制：
MyISAM引擎：
Myisam引擎允许在使用分区时，把表的不同部分存储在不同地方，包括索引目录和数据目录。
下面是一个关于把数据分布到4个不同的物理磁盘上的myisam分区。
Create table orders_hash2
(
Id int auto_increment primary key, ……
) engine=myisam

Partition by hash(id)
(
Partition p0 index directory=’/data0/orders/idx’
data directory=’ /data0/orders/data’,
Partition p1 index directory=’/data1/orders/idx’
data directory=’ /data1/orders/data’,
Partition p2 index directory=’/data2/orders/idx’
data directory=’ /data2/orders/data’,
Partition p3 index directory=’/data3/orders/idx’
data directory=’ /data3/orders/data’,
);

注意：上面的具体4个分布，在windows系统上目前还不支持。

InnoDB引擎：
Innodb的分区管理与myisam引擎的管理是不同的。


分区的限制性
下面讲到的是一些关于MySQL分区的限制性约束
常见的限制：
所有的分区必须使用同种引擎；
批量装载很慢；
每个表的最大分区数为1024；
不支持三维数据类型(GIS);
不能对临时表进行分区；
不可能对日志表进行分区；

外键和索引方面：
不支持外键；
不支持全文表索引；
不支持load cache和load index into cache；

子分区方面：
只允许对range和list类型的分区再进行分区；
子分区的类型只允许是hash或key.

分区表达式方面：
Range，list, hash分区必须是int类型；
Key分区不可以有text，blob类型；
不允许使用UDF,存储函数，变量，操作符(|,,^,<<,>>,~)和一些内置的函数；
在表创建之后sql mode不可以改变；
在分区表达式中，不允许子查询；
分区表达式中必须包括至少一个列的引用，唯一索引列也可以(包括主键)

===================================================
Mysql分区表局限性总结

Mysql5.1已经发行很久了，本文根据官方文档的翻译和自己的一些测试，对Mysql分区表的局限性做了一些总结，因为个人能力以及测试环境的原因，有可能有错误的地方，还请大家看到能及时指出，当然有兴趣的朋友可以去官方网站查阅。

本文测试的版本

mysql> select version();
+------------+
| version()  |
+------------+
| 5.1.33-log | 
+------------+
1 row in set (0.00 sec)
一、关于Partitioning Keys, Primary Keys, and Unique Keys的限制

在5.1中分区表对唯一约束有明确的规定，每一个唯一约束必须包含在分区表的分区键（也包括主键约束）。
这句话也许不好理解，我们做几个实验：

CREATE TABLE t1   
(      id INT NOT NULL,      
       uid INT NOT NULL,
       PRIMARY KEY (id)
)
PARTITION BY RANGE (id)   
(PARTITION p0 VALUES LESS THAN(5) ENGINE = INNODB,
 PARTITION p1 VALUES LESS THAN(10) ENGINE = INNODB
);
 
CREATE TABLE t1   
(      id INT NOT NULL,      
       uid INT NOT NULL,
       PRIMARY KEY (id)
)
PARTITION BY RANGE (id)   
(PARTITION p0 VALUES LESS THAN(5) ENGINE = MyISAM DATA DIRECTORY='/tmp' INDEX DIRECTORY='/tmp',
 PARTITION p1 VALUES LESS THAN(10) ENGINE = MyISAM DATA DIRECTORY='/tmp' INDEX DIRECTORY='/tmp'
);
 
mysql> CREATE TABLE t1   
    -> (      id INT NOT NULL,      
    ->        uid INT NOT NULL,
    ->        PRIMARY KEY (id),
    ->        UNIQUE KEY (uid)
    -> )
    -> PARTITION BY RANGE (id)   
    -> (PARTITION p0 VALUES LESS THAN(5),
    ->  PARTITION p1 VALUES LESS THAN(10)
    -> );
ERROR 1503 (HY000): A UNIQUE INDEX must include all columns in the table's partitioning function
二、关于存储引擎的限制
2.1 MERGE引擎不支持分区，分区表也不支持merge。
2.2 FEDERATED引擎不支持分区。这限制可能会在以后的版本去掉。
2.3 CSV引擎不支持分区
2.4 BLACKHOLE引擎不支持分区
2.5 在NDBCLUSTER引擎上使用分区表，分区类型只能是KEY(or LINEAR KEY) 分区。
2.6 当升级MYSQL的时候，如果你有使用了KEY分区的表（不管是什么引擎，NDBCLUSTER除外），那么你需要把这个表dumped在reloaded。
2.7 分区表的所有分区或者子分区的存储引擎必须相同，这个限制也许会在以后的版本取消。
不指定任何引擎（使用默认引擎）。
所有分区或者子分区指定相同引擎。

三、关于函数的限制
在mysql5.1中建立分区表的语句中，只能包含下列函数：
ABS()
CEILING() and FLOOR() （在使用这2个函数的建立分区表的前提是使用函数的分区键是INT类型），例如

mysql> CREATE TABLE t (c FLOAT) PARTITION BY LIST( FLOOR(c) )(
    -> PARTITION p0 VALUES IN (1,3,5),
    -> PARTITION p1 VALUES IN (2,4,6)
    -> );;
ERROR 1491 (HY000): The PARTITION function returns the wrong type
 
mysql> CREATE TABLE t (c int) PARTITION BY LIST( FLOOR(c) )(
    -> PARTITION p0 VALUES IN (1,3,5),
    -> PARTITION p1 VALUES IN (2,4,6)
    -> );
Query OK, 0 rows affected (0.01 sec)
DAY()
DAYOFMONTH()
DAYOFWEEK()
DAYOFYEAR()
DATEDIFF()
EXTRACT()
HOUR()
MICROSECOND()
MINUTE()
MOD()
MONTH()
QUARTER()
SECOND()
TIME_TO_SEC()
TO_DAYS()
WEEKDAY()
YEAR()
YEARWEEK()

四、其他限制

4.1 对象限制
下面这些对象在不能出现在分区表达式
Stored functions, stored procedures, UDFs, or plugins.
Declared variables or user variables.

4.2 运算限制
支持加减乘等运算出现在分区表达式，但是运算后的结果必须是一个INT或者NULL。 |, &, ^, <<, >>, , ~ 等不允许出现在分区表达式。

4.3 sql_mode限制
官方强烈建议你在创建分区表后，永远别改变mysql的sql_mode。因为在不同的模式下，某些函数或者运算返回的结果可能会不一样。

4.4 Performance considerations.(省略)

4.5 最多支持1024个分区，包括子分区。
当你建立分区表包含很多分区但没有超过1024限制的时候，如果报错 Got error 24 from storage engine，那意味着你需要增大open_files_limit参数。

4.6 不支持外键。MYSQL中，INNODB引擎才支持外键。

4.7 不支持FULLTEXT indexes（全文索引），包括MYISAM引擎。

mysql> CREATE TABLE articles (
    -> id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    -> title VARCHAR(200),
    -> body TEXT,
    -> FULLTEXT (title,body)
    -> )
    -> PARTITION BY HASH(id)
    -> PARTITIONS 4;
ERROR 1214 (HY000): The used table type doesn't support FULLTEXT indexes
4.8 不支持spatial column types。
4.9 临时表不能被分区。

mysql> CREATE Temporary TABLE t1   
    -> (      id INT NOT NULL,      
    ->        uid INT NOT NULL,
    ->        PRIMARY KEY (id)
    -> )
    -> PARTITION BY RANGE (id)   
    -> (PARTITION p0 VALUES LESS THAN(5) ENGINE = MyISAM,
    ->  PARTITION p1 VALUES LESS THAN(10) ENGINE = MyISAM
    -> );
ERROR 1562 (HY000): Cannot create temporary table with partitions
4.10 log table不支持分区。

mysql> alter table mysql.slow_log PARTITION BY KEY(start_time) PARTITIONS 2;
ERROR 1221 (HY000): Incorrect usage of PARTITION and log table
5.11 分区键必须是INT类型，或者通过表达式返回INT类型，可以为NULL。唯一的例外是当分区类型为KEY分区的时候，可以使用其他类型的列作为分区键（ BLOB or TEXT 列除外）。

mysql> CREATE TABLE tkc (c1 CHAR)
    -> PARTITION BY KEY(c1)
    -> PARTITIONS 4;
Query OK, 0 rows affected (0.00 sec)
 
mysql> CREATE TABLE tkc2 (c1 CHAR)
    -> PARTITION BY HASH(c1)
    -> PARTITIONS 4;
ERROR 1491 (HY000): The PARTITION function returns the wrong type
 
mysql> CREATE TABLE tkc3 (c1 INT)
    -> PARTITION BY HASH(c1)
    -> PARTITIONS 4;
Query OK, 0 rows affected (0.00 sec)
5.12 分区键不能是一个子查询。 A partitioning key may not be a subquery, even if that subquery resolves to an integer value or NULL

5.13 只有RANG和LIST分区能进行子分区。HASH和KEY分区不能进行子分区。

5.14 分区表不支持Key caches。

mysql> SET GLOBAL keycache1.key_buffer_size=128*1024;
Query OK, 0 rows affected (0.00 sec)
mysql> CACHE INDEX login,user_msg,user_msg_p IN keycache1;
+-----------------+--------------------+----------+---------------------------------------------------------------------+
| Table           | Op                 | Msg_type | Msg_text                                                            |
+-----------------+--------------------+----------+---------------------------------------------------------------------+
| test.login      | assign_to_keycache | status   | OK                                                                  | 
| test.user_msg   | assign_to_keycache | status   | OK                                                                  | 
| test.user_msg_p | assign_to_keycache | note     | The storage engine for the table doesn't support assign_to_keycache | 
+-----------------+--------------------+----------+---------------------------------------------------------------------+
3 rows in set (0.00 sec)
5.15 分区表不支持INSERT DELAYED.

mysql> insert  DELAYED into user_msg_p values(18156629,0,0,0,0,0,0,0,0,0);
ERROR 1616 (HY000): DELAYED option not supported for table 'user_msg_p'
5.16 DATA DIRECTORY 和 INDEX DIRECTORY 参数在分区表将被忽略。
这个限制应该不存在了：

mysql> CREATE TABLE t1   
    -> (      id INT NOT NULL,      
    ->        uid INT NOT NULL,
    ->        PRIMARY KEY (id)
    -> )
    -> PARTITION BY RANGE (id)   
    -> (PARTITION p0 VALUES LESS THAN(5) ENGINE = MyISAM DATA DIRECTORY='/tmp' INDEX DIRECTORY='/tmp',
    ->  PARTITION p1 VALUES LESS THAN(10) ENGINE = MyISAM DATA DIRECTORY='/tmp' INDEX DIRECTORY='/tmp'
    -> );
Query OK, 0 rows affected (0.01 sec)
5.17 分区表不支持mysqlcheck和myisamchk
在5.1.33版本中已经支持mysqlcheck和myisamchk

./mysqlcheck -u -p -r test user_msg_p;
test.user_msg_p                                    OK
 
./myisamchk -i /u01/data/test/user_msg_p#P#p0.MYI
Checking MyISAM file: /u01/data/test/user_msg_p#P#p0.MYI
Data records: 4423615   Deleted blocks:       0
- check file-size
- check record delete-chain
- check key delete-chain
- check index reference
- check data record references index: 1
Key:  1:  Keyblocks used:  98%  Packed:    0%  Max levels:  4
Total:    Keyblocks used:  98%  Packed:    0%
 
User time 0.97, System time 0.02
Maximum resident set size 0, Integral resident set size 0
Non-physical pagefaults 324, Physical pagefaults 0, Swaps 0
Blocks in 0 out 0, Messages in 0 out 0, Signals 0
Voluntary context switches 1, Involuntary context switches 5
5.18 分区表的分区键创建索引，那么这个索引也将被分区。分区键没有全局索引一说。
5.19 在分区表使用ALTER TABLE … ORDER BY，只能在每个分区内进行order by。


=================================================

MySQL 分区(Partition)脚本

MySQL 5.1 中新特性分区(partition) shell 脚本。注意 MySQL 只支持小于等于 1024 个分区。

 

#!/bin/sh


# Set these values
PART=0
ORI=5000
STEP=5000
MAX=3000000

for NUM in `seq -f %f $ORI $STEP $MAX | cut -d. -f1`
do
        echo “PARTITION $PART VALUES LESS THAN ($NUM),” >> /tmp/partition.sql
        part=`expr $PART + 1`
done
echo “PARTITION $PART VALUES LESS THAN MAXVALUE >> /tmp/partition.sql



############################################################################
重建分区: 这和先删除保存在分区中的所有记录，然后重新插入它们，具有同样的效果。它可用于整理分区碎片。 
ALTER TABLE t1 REBUILD PARTITION (p0, p1)；

优化分区：如果从分区中删除了大量的行，或者对一个带有可变长度的行（也就是说，有VARCHAR，BLOB，或TEXT类型的列）作了许多修改，可以使用“ALTER TABLE ... OPTIMIZE PARTITION”来收回没有使用的空间，并整理分区数据文件的碎片。
ALTER TABLE t1 OPTIMIZE PARTITION (p0, p1)；

分析分区：读取并保存分区的键分布。
ALTER TABLE t1 ANALYZE PARTITION (p3)；

修补分区： 修补被破坏的分区。 
ALTER TABLE t1 REPAIR PARTITION (p0,p1);

检查分区： 可以使用几乎与对非分区表使用CHECK TABLE 相同的方式检查分区。
ALTER TABLE trb3 CHECK PARTITION (p1)；
```