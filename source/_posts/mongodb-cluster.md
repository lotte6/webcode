---
title: MongoDB 分片集群部署
categories: tech
tag: hide
date: 2018-12-26 10:37:36
tags:
---
### MongoDB分片（Sharding）技术
　　分片（sharding）是MongoDB用来将大型集合分割到不同服务器（或者说一个集群）上所采用的方法。尽管分片起源于关系型数据库分区，但MongoDB分片完全又是另一回事。  
　　和MySQL分区方案相比，MongoDB的最大区别在于它几乎能自动完成所有事情，只要告诉MongoDB要分配数据，它就能自动维护数据在不同服务器之间的均衡。  

1. 分片的目的
　　高数据量和吞吐量的数据库应用会对单机的性能造成较大压力,大的查询量会将单机的CPU耗尽,大的数据量对单机的存储压力较大,最终会耗尽系统的内存而将压力转移到磁盘IO上。
　　为了解决这些问题,有两个基本的方法: 垂直扩展和水平扩展。
　　　　垂直扩展：增加更多的CPU和存储资源来扩展容量。
　　　　水平扩展：将数据集分布在多个服务器上。水平扩展即分片。

3. 分片设计思想
　　分片为应对高吞吐量与大数据量提供了方法。使用分片减少了每个分片需要处理的请求数，因此，通过水平扩展，集群可以提高自己的存储容量和吞吐量。举例来说，当插入一条数据时，应用只需要访问存储这条数据的分片。　　
　　使用分片减少了每个分片存储的数据。　　
　　例如，如果数据库3tb的数据集，并有4个分片，然后每个分片可能仅持有356 GB的数据。如果有40个分片，那么每个切分可能只有35GB的数据。
</br>
### 分片机制提供了如下三种优势
3. 对集群进行抽象，让集群“不可见”  
　　MongoDB自带了一个叫做mongos的专有路由进程。mongos就是掌握统一路口的路由器，其会将客户端发来的请求准确无误的路由到集群中的一个或者一组服务器上，同时会把接收到的响应拼装起来发回到客户端。  
3. 保证集群总是可读写  
　　MongoDB通过多种途径来确保集群的可用性和可靠性。将MongoDB的分片和复制功能结合使用，在确保数据分片到多台服务器的同时，也确保了每分数据都有相应的备份，这样就可以确保有服务器换掉时，其他的从库可以立即接替坏掉的部分继续工作。
3. 使集群易于扩展  
　　当系统需要更多的空间和资源的时候，MongoDB使我们可以按需方便的扩充系统容量。

组件 说明
Config Server 存储集群所有节点、分片数据路由信息。默认需要配置3个Config Server节点。
Mongos 提供对外应用访问，所有操作均通过mongos执行。一般有多个mongos节点。数据迁移和数据自动平衡。
Mongod 存储应用数据记录。一般有多个Mongod节点，达到数据分片目的。


组件 | 说明 
---------|----------
 Config Server | 存储集群所有节点、分片数据路由信息。默认需要配置3个Config Server节点。 
 Mongos | 提供对外应用访问，所有操作均通过mongos执行。一般有多个mongos节点。数据迁移和数据自动平衡。 
 Mongod | 存储应用数据记录。一般有多个Mongod节点，达到数据分片目的。 

</br>
### 分片集群的构造
   3. mongos ：数据路由，和客户端打交道的模块。mongos本身没有任何数据，他也不知道该怎么处理这数据，去找config server
   3. config server：所有存、取数据的方式，所有shard节点的信息，分片功能的一些配置信息。可以理解为真实数据的元数据。
   3. shard：真正的数据存储位置，以chunk为单位存数据。
　　Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会议分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。

Mongos的路由功能
　　当数据写入时，MongoDB Cluster根据分片键设计写入数据。
　　当外部语句发起数据查询时，MongoDB根据数据分布自动路由至指定节点返回数据。
   3. mongos ：数据路由，和客户端打交道的模块。mongos本身没有任何数据，他也不知道该怎么处理这数据，去找config server
   3. config server：所有存、取数据的方式，所有shard节点的信息，分片功能的一些配置信息。可以理解为真实数据的元数据。
   3. shard：真正的数据存储位置，以chunk为单位存数据。
　　Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会议分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。


</br>
 ## Chunk
　　在一个shard server内部，MongoDB还是会把数据分为chunks，每个chunk代表这个shard server内部一部分数据。chunk的产生，会有以下两个用途：
　　Splitting：当一个chunk的大小超过配置中的chunk size时，MongoDB的后台进程会把这个chunk切分成更小的chunk，从而避免chunk过大的情况
　　Balancing：在MongoDB中，balancer是一个后台进程，负责chunk的迁移，从而均衡各个shard server的负载，系统初始3个chunk，chunk size默认值64M,生产库上选择适合业务的chunk size是最好的。ongoDB会自动拆分和迁移chunks。
分片集群的数据分布（shard节点）
   3. 使用chunk来存储数据
   3. 进群搭建完成之后，默认开启一个chunk，大小是64M，
   3. 存储需求超过64M，chunk会进行分裂，如果单位时间存储需求很大，设置更大的chunk
   4. chunk会被自动均衡迁移。
### chunksize的选择
　　适合业务的chunksize是最好的。
　　chunk的分裂和迁移非常消耗IO资源；chunk分裂的时机：在插入和更新，读数据不会分裂。
　　chunksize的选择：
　　小的chunksize：数据均衡是迁移速度快，数据分布更均匀。数据分裂频繁，路由节点消耗更多资源。大的chunksize：数据分裂少。数据块移动集中消耗IO资源。通常300-300M
### chunk分裂及迁移
　　随着数据的增长，其中的数据大小超过了配置的chunk size，默认是64M，则这个chunk就会分裂成两个。数据的增长会让chunk分裂得越来越多。
这时候，各个shard 上的chunk数量就会不平衡。这时候，mongos中的一个组件balancer  就会执行自动平衡。把chunk从chunk数量最多的shard节点挪动到数量最少的节点。
chunkSize 对分裂及迁移的影响
　　MongoDB 默认的 chunkSize 为64MB，如无特殊需求，建议保持默认值；chunkSize 会直接影响到 chunk 分裂、迁移的行为。
　　chunkSize 越小，chunk 分裂及迁移越多，数据分布越均衡；反之，chunkSize 越大，chunk 分裂及迁移会更少，但可能导致数据分布不均。
　　chunkSize 太小，容易出现 jumbo chunk（即shardKey 的某个取值出现频率很高，这些文档只能放到一个 chunk 里，无法再分裂）而无法迁移；chunkSize 越大，则可能出现 chunk 内文档数太多（chunk 内文档数不能超过 350000 ）而无法迁移。
　　chunk 自动分裂只会在数据写入时触发，所以如果将 chunkSize 改小，系统需要一定的时间来将 chunk 分裂到指定的大小。
　　chunk 只会分裂，不会合并，所以即使将 chunkSize 改大，现有的 chunk 数量不会减少，但 chunk 大小会随着写入不断增长，直到达到目标大小。
</br>
## 数据区分
### 分片键shard key
　　MongoDB中数据的分片是、以集合为基本单位的，集合中的数据通过片键（Shard key）被分成多部分。其实片键就是在集合中选一个键，用该键的值作为数据拆分的依据。
　　所以一个好的片键对分片至关重要。片键必须是一个索引，通过sh.shardCollection加会自动创建索引（前提是此集合不存在的情况下）。一个自增的片键对写入和数据均匀分布就不是很好，因为自增的片键总会在一个分片上写入，后续达到某个阀值可能会写到别的分片。但是按照片键查询会非常高效。
　　随机片键对数据的均匀分布效果很好。注意尽量避免在多个分片上进行查询。在所有分片上查询，mongos会对结果进行归并排序。
　　对集合进行分片时，你需要选择一个片键，片键是每条记录都必须包含的，且建立了索引的单个字段或复合字段，MongoDB按照片键将数据划分到不同的数据块中，并将数据块均衡地分布到所有分片中。
　　为了按照片键划分数据块，MongoDB使用基于范围的分片方式或者 基于哈希的分片方式。
注意：
分片键是不可变。
分片键必须有索引。
分片键大小限制533bytes。
分片键用于路由查询。
MongoDB不接受已进行collection级分片的collection上插入无分片
键的文档（也不支持空值插入）
</br>
### 以范围为基础的分片Sharded Cluster
　　Sharded Cluster支持将单个集合的数据分散存储在多shard上，用户可以指定根据集合内文档的某个字段即shard key来进行范围分片（range sharding）。
　　对于基于范围的分片，MongoDB按照片键的范围把数据分成不同部分。
　　假设有一个数字的片键:想象一个从负无穷到正无穷的直线，每一个片键的值都在直线上画了一个点。MongoDB把这条直线划分为更短的不重叠的片段，并称之为数据块，每个数据块包含了片键在一定范围内的数据。在使用片键做范围划分的系统中，拥有”相近”片键的文档很可能存储在同一个数据块中，因此也会存储在同一个分片中。

## 设计
3. 使用3台机器，每台有独立的ip地址，并且在同一网段内
3. 每台机器运行所有服务，即：mongos、config、shard3、shard3、shard3
3. 容器化所有服务
![mongodb](/img/mongodb.jpg)

## 部署
3.期初测试的时候是直接yum安装MongoDB，官方安装文档：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/  
安装过程很简单，就是添加官方源即可。以下操作仅做参考，纯手写，可能有些不能直接复制执行。切记切记。
```
cat >>/etc/yum.repos.d/mongodb-org-4.0.repo <<EOF
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=3
enabled=3
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
EOF

yum install -y mongodb-org
```

3. 最后的调试结果是在rancher平台上直接以宿主机模式启动各个服务，所以不需要在机器上安装MongoDB服务，只依赖Docker，Docker的安装就不再赘述，拉去官方的镜像
```
docker pull mongo:4.0.3-xenial

当时的最新稳定版
```
3. 启动  
命令行执行如下，如果要启动在docker管理平台即在命令里执行-c后面的命令即可

* shard3启动：  
docker run -d mongo:4.0.3-xenial -c "mongod –shardsvr –replSet shard3 –bind_ip_all –port 37003 –dbpath /data/mongodb/shard3/data –logpath /data/mongodb/shard3/log/shard3.log –fork –oplogSize 30 –wiredTigerCacheSizeGB 8G"

* shard3启动： 
docker run -d mongo:4.0.3-xenial -c "mongod –shardsvr –replSet shard2 –bind_ip_all –port 37002 –dbpath /data/mongodb/shard2/data –logpath /data/mongodb/shard2/log/shard2.log –fork –oplogSize 30 –wiredTigerCacheSizeGB 8G"
* shard3启动： 
docker run -d mongo:4.0.3-xenial -c "mongod –shardsvr –replSet shard3 –bind_ip_all –port 37001 –dbpath /data/mongodb/shard1/data –logpath /data/mongodb/shard3/log/shard1.log –fork –oplogSize 30 –wiredTigerCacheSizeGB 8G"
* config启动：
docker run -d mongo:4.0.3-xenial -c "mongod –configsvr –replSet myset –bind_ip_all –dbpath /data/mongodb/config/data –port 27000 –logpath /data/mongodb/config/log/config.log –fork –wiredTigerCacheSizeGB 1G"
* mongos启动： 
docker run -d mongo:4.0.3-xenial -c "mongos –configdb myset/172.20.101.132:27000,172.20.101.133:27000,172.20.101.134:27000 –port 27017 –logpath /data/mongodb/mongos/mongos.log"
4. 设置副本集
链接shard1并执行：  
mongo  127.0.0.1:27001/admin
```
config = { _id:"shard1", members:[
                     {_id:0,host:"172.20.101.132:27001"},
                     {_id:1,host:"172.20.101.133:27001"},
                     {_id:2,host:"172.20.101.134:27001",arbiterOnly:true}
                ]
         }
rs.initiate(config) 
```

shard2、shard3、config依次执行(注意链接时候的端口)

```
config = { _id:"shard2", members:[
                     {_id:0,host:"172.20.101.132:27002"},
                     {_id:1,host:"172.20.101.133:27002"},
                     {_id:2,host:"172.20.101.134:27002",arbiterOnly:true}
                ]
         }
rs.initiate(config)		 
config = { _id:"shard3", members:[
                     {_id:0,host:"172.20.101.132:27003"},
                     {_id:1,host:"172.20.101.133:27003"},
                     {_id:2,host:"172.20.101.134:27003",arbiterOnly:true}
                ]
         }
rs.initiate(config)


#config server 副本
config = {_id: 'myset', members: [
                          {_id: 0, host: '172.20.101.132:27000'},
                          {_id: 1, host: '172.20.101.133:27000'},
                          {_id: 2, host: '172.20.101.134:27000'}]
           }
rs.initiate(config)
```

5. 登录mongos添加节点  
mongo  127.0.0.1:27017/admin
```
db.runCommand( { addshard : "shard1/172.20.101.132:27001,172.20.101.133:27001,172.20.101.134:27001",name:"shard1"});
db.runCommand( { addshard : "shard2/172.20.101.132:27002,172.20.101.133:27002,172.20.101.134:27002",name:"shard2"});
db.runCommand( { addshard : "shard3/172.20.101.132:27003,172.20.101.133:27003,172.20.101.134:27003",name:"shard3"});
```

6. 查看分片状态
```
db.runCommand( { listshards : 1 } ) #列出分片  
mongos> sh.status();
```

至此数据库分片集群部署完成，后续更新管理相关经验技术点。

## 维护

### 基础命令

1. 查看分片  
db.shards.find('shard1')  
2. 查看分片集合信息  
db.databases.find()  
3. 查看所有mongos状态
db.collections.findOne()  
4. 判断是否Shard集群  
db.runCommand({ isdbgrid : 1})  
5. 列出所有分片信息  
admin> db.runCommand({ listshards : 1})  
6. 冻结其中的一个从节点，使其不参与到与primary 的内部选举工作，进入客户端,执行(单位:秒)
rs.freeze(30)  
7. 对原主节点进行降级，进入客户端,执行下面代码 (单位:秒)
rs.stepDown(15)  
8. 经过冻结和降级之后查看复制集状态 
rs.status()
9. MongoDB数据备份  
```
mongodump -h dbhost -d dbname -o dbdirectory
-h：
MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
-d：
需要备份的数据库实例，例如：test
-o：
备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。
```

10. MongoDB数据恢复
```
mongorestore -h <hostname><:port> -d dbname <path>
--host  
MongoDB所在服务器地址，默认为： localhost:27017
--db  
需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2
--drop：
恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！
<path>：
mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。
你不能同时指定 <path> 和 --dir 选项，--dir也可以设置备份目录。
--dir：
指定备份的目录
你不能同时指定 <path> 和 --dir 选项。
```
