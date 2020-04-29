---
title: mysql_optimization
categories: mysql
tag: hide
date: 2018-12-24 11:33:08
tags:
---
```
看了一下，其实此查询嵌套用得不好，作为程序员应该尽量避免用 not in ，in,  left join  ,right  join 等等，不过这些不归我管，我只能提一些建议。
（顺便说一声：oracle里面可以用 exist ,not exist, minus等代替in ,not in  效率高出很多 ）
此分析对我没有太大的作用，因此用google查询了一下，发现网上一篇文章讲得很好，
（ http://clay111.blogchina.com/4721079.html 我给转贴了，感兴趣可以看看）
Copying to tmp table on disk The temporary result set was larger than tmp_table_size and the 
thread is now changing the in memory-based temporary table to a disk based one to save memory. 
哦，原来是这样的，如果查询超出了tmp_table_size的限制，那么mysql用/tmp保存查询结果，然后返回给客户端。
set global tmp_table_size=209715200  (200M)

再次运行此查询,用/opt/mysql/bin/mysqladmin processlist; 进行观察,发现不会出现上述问题.
至此问题解决.

调节tmp_table_size  的时候发现另外一些参数
Qcache_queries_in_cache  在缓存中已注册的查询数目  
Qcache_inserts  被加入到缓存中的查询数目  
Qcache_hits  缓存采样数数目  
Qcache_lowmem_prunes  因为缺少内存而被从缓存中删除的查询数目  
Qcache_not_cached  没有被缓存的查询数目 (不能被缓存的，或由于 QUERY_CACHE_TYPE)  
Qcache_free_memory  查询缓存的空闲内存总数  
Qcache_free_blocks  查询缓存中的空闲内存块的数目  
Qcache_total_blocks  查询缓存中的块的总数目  

Qcache_free_memory 可以缓存一些常用的查询,如果是常用的sql会被装载到内存。那样会增加数据库访问速度。

sort_buffer_size
max_length_for_sort_data这个值，增大到8096

set global concurrent_insert = 2;
通常来说，在MyISAM里读写操作是串行的，但当对同一个表进行查询和插入操作时，为了降低锁竞争的频率，根据concurrent_insert的设置，MyISAM是可以并行处理查询和插入的：
当concurrent_insert=0时，不允许并发插入功能。
当concurrent_insert=1时，允许对没有洞洞的表使用并发插入，新数据位于数据文件结尾（缺省）。
当concurrent_insert=2时，不管表有没有洞洞，都允许在数据文件结尾并发插入。
这样看来，把concurrent_insert设置为2是很划算的，至于由此产生的文件碎片，可以定期使用OPTIMIZE TABLE语法优化。

myisam_max_sort_file_size=20G
sort_buffer_size=20G

max_allowed_packet      = 16M
key_buffer              = 2G
tmp_table_size          = 1G
sort_buffer_size        = 1G
query_cache_size        = 512M
query_cache_limit       = 32M
query_cache_type        = 1
max_length_for_sort_data = 8M
join_buffer_size        = 8M
wait_timeout            = 60
interactive_timeout     = 60
```