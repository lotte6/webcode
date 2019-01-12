---
title: Amazon AWS 中国区的那些坑
categories: tech
tag: hide
date: 2018-12-18 22:09:03
tags:
---


使用AWS 中国区有一段时间了, 期间踩过了一些坑. 简单写一下, 希望对别人有帮助。文中很可能有错误或者AWS 已经升级了, 还是用他们的 support 服务最靠谱。
Amazon S3
所有坑中, 最数 S3 坑多. 原因很简单: EC2的服务大不了大家在web console 里面点击鼠标, S3 更多时候肯定是用SDK访问. 因此SDK的各种问题都会提前暴露.
Hadoop Over S3
问题: 去年12月份左右(具体jets3t 什么时候fix的这个问题不记得了), hadoop 中使用的library jets3t 不支持中国区(cn-north-1) , 原因很简单: S3 的signature 已经升级到V4. 但是因为兼容问题, AWS的其他region都兼容V2版本, 中国区是新的region, 没有兼容问题, 因此仅仅支持V4. 详情参见 jets3t 的这个issue
折腾了各种解决办法, 流水账的形式写一下吧.
第一个法子: copy EMR 集群中的emrfs
emrfs 就是 EMR 集群中hadoop使用的访问S3 的方式. 是 Amazon
官方提供的, 不开源. 使用的法子也很简单: 启动一个emr 集群, 随便登陆一台服务器, 在 hadoop-env.sh 中可以看到添加了emrfs 的classpath:
```
#!/bin/bashexport HADOOP_CLIENT_OPTS="$HADOOP_CLIENT_OPTS -XX:MaxPermSize=128m"export HADOOP_CLASSPATH="$HADOOP_CLASSPATH:/usr/share/aws/emr/emrfs/lib/*:/usr/share/aws/emr/lib/*"export HADOOP_DATANODE_HEAPSIZE="384"export HADOOP_NAMENODE_HEAPSIZE="768"export HADOOP_OPTS="$HADOOP_OPTS -server"if [ -e /home/hadoop/conf/hadoop-user-env.sh ] ; then  . /home/hadoop/conf/hadoop-user-env.shfi
```
注意: EMR 可能会发布新的版本, 这里仅仅是提供一个思路, 列出的文件也是当时版本的emr的实现
将 /usr/share/aws/emr/emrfs 下面的所有文件copy出来, 部署到自己的集群并在 core-sites.xml 中添加如下内容:
```
fs.s3n.implcom.amazon.ws.emr.hadoop.fs.EmrFileSystemfs.s3.implcom.amazon.ws.emr.hadoop.fs.EmrFileSystemfs.s3.buffer.dir/mnt/var/lib/hadoop/s3,/mnt1/var/lib/hadoop/s3fs.s3.buckets.create.regioncn-north-1fs.s3bfs.implorg.apache.hadoop.fs.s3.S3FileSystemfs.s3n.endpoints3.cn-north-1.amazonaws.com.cn
```
设置 EMRFS_HOME 并且把 $EMRFS_HOME/bin 添加到PATH中(后面会用到)
这样可以保证hadoop 尽快运行起来. 但使用 emrfs 也有一些问题:
没有源代码. 官方没有计划将这个东西开源. 因此除了问题只有反编译jar包. 还好官方编译的jar包没有混淆并且带着 lineNumber 等信息. 曾经遇到他代码里面吃掉异常的情况, 不知道现在是否更新

S3 rename 操作非常耗时. 众所周知Hadoop Mapreduce 为了保证一致性, 结果文件都是先写临时文件, 最后 rename 成最终输出文件. 在 HDFS 上这种模式没有问题, 但是 S3 就会导致最后 commit job 时非常慢, 因此默认的committer 是单线程rename文件. 结果文件大并且多文件的情况下S3 非常慢. 因此 emrfs 做了一个hack: 结果仅仅写本地文件, 到 commit 的时候再一次性上传结果文件. 但如果你输出的一个结果文件太大会导致本地磁盘写满! 不知道哪里是否有参数配置一下这个最大值.

S3 由于不是FileSystem, 仅仅是一个KV存储. 因此在list dir 时会很慢, emrfs 为了优化, 用dynamodb做了一层索引.但在某些情况下(我们遇到过)mr job fail 会导致索引和 S3 数据不一致. 极端情况下需要使用 emrfs sync path 来同步索引
暂时记得的关于 emrfs 就有这么多.

第二个法子: hadoop-s3a
An AWS SDK-backed FileSystem driver for Hadoop
这是github上有人用 AWS-java-SDK 开发的一个 FileSystem 实现, 虽说是试验情况下, 修改一下还是可以用的. >;<
但是, 这个直接用也是不行的!~~~
坑如下:
中国区 Amazon S3 Java SDK 有一个神坑: 如果不显示设置region的 endpoint , 会一直返回 Invalid Request(原因后面解释), 需要在代码中添加如下几行:
// 这里获取配置文件中的region name的设置//  如果获取不到, 强烈建议获取当前系统所在
```
regionAmazonS3Client s3 = new AmazonS3Client(credentials, awsConf);String regionName = XXXX;Region region = Region.getRegion(Regions.fromName(regionName));s3.setRegion(region);final String serviceEndpoint = region.getServiceEndpoint(ServiceAbbreviations.S3);// 关键是下面这一行, 在除了中国外的其他region, 这行代码不用写s3.setEndpoint(serviceEndpoint);LOG.info("setting s3 region: " + region + ", : " + serviceEndpoi
```
S3 rename 操作慢!
需要在 hadoop-s3a 中需要修改rename 方法的代码, 使用线程池并行rename 一个dir.
需要写一个 committer, 在MR job 完成的时候调用并行rename操作.
hadoop-s3a 没有设置 connect timeout. 仅仅设置了socket timetout
block size计算错误.
需要在社区版本上添加一个 block size 的配置项(跟 hdfs 类似), 并且在所有创建 S3AFileStatus 的地方添加 blockSize 参数. 现在情况下会导致计算 InputSplit 错误(blocksize默认是0).
权限管理
通常情况下, hadoop 集群使用IAM role 方式获取accessKey 访问S3, 这样会导致之前在 hdfs 中基于用户的权限管理失效. 比如, 用户A 是对一些Table 有读写权限, 但是用户B 只有只读权限. 但EC2 不支持一个instance 挂载两个不同的 IAM role. 我们的解决方案是在S3FileSystem中判断当前的用户, 根据不同的用户使用不同的AccessKey, 实现HDFS的权限管理.
S3 api/client
使用S3 api 或者aws client, 还有一个容易误导的坑:
你有可能在 cn-north-1 的region 访问到AWS 美国的S3 !
现象: 比如你按照doc 配置好了aws client(access key 和secret都配置好), 简单执行 
```
aws --debug s3 ls s3://your-bucket/ 确返回如下错误:
2015-08-06 20:54:25,622 - botocore.endpoint - DEBUG - Sending http request:  2015-08-06 20:54:27,770 - botocore.response - DEBUG - Response Body: b'\nInvalidAccessKeyIdThe AWS Access Key Id you provided does not exist in our records.AAABBBBAAAAAA111B1ABCFEA8D30AfPehbRNkUmZyI6/O3kL7s+J0zCLYo/8U6UE+Hv7PSBFiA6cB6CuLXoCT4pvyiO7l' 2015-08-06 20:54:27,783 - botocore.hooks - DEBUG - Event needs-retry.s3.ListObjects: calling handler  2015-08-06 20:54:27,783 - botocore.retryhandler - DEBUG - No retry needed. 2015-08-06 20:54:27,784 - botocore.hooks - DEBUG - Event after-call.s3.ListObjects: calling handler  2015-08-06 20:54:27,784 - awscli.errorhandler - DEBUG - HTTP Response Code: 403 2015-08-06 20:54:27,784 - awscli.clidriver - DEBUG - Exception caught in main() Traceback (most recent call last):   File "/usr/share/awscli/awscli/clidriver.py", line 187, in main     return command_table[parsed_args.command](remaining, parsed_args)   File "/usr/share/awscli/awscli/customizations/s3/s3.py", line 165, in __call__     remaining, parsed_globals)   File "/usr/share/awscli/awscli/customizations/s3/s3.py", line 276, in __call__     return self._do_command(parsed_args, parsed_globals)   File "/usr/share/awscli/awscli/customizations/s3/s3.py", line 358, in _do_command     self._list_all_objects(bucket, key)   File "/usr/share/awscli/awscli/customizations/s3/s3.py", line 365, in _list_all_objects     for _, response_data in iterator:   File "/usr/lib/python3/dist-packages/botocore/paginate.py", line 147, in __iter__     **current_kwargs)   File "/usr/lib/python3/dist-packages/botocore/operation.py", line 82, in call     parsed=response[1])   File "/usr/lib/python3/dist-packages/botocore/session.py", line 551, in emit     return self._events.emit(event_name, **kwargs)   File "/usr/lib/python3/dist-packages/botocore/hooks.py", line 158, in emit     response = handler(**kwargs)   File "/usr/share/awscli/awscli/errorhandler.py", line 75, in __call__     http_status_code=http_response.status_code) awscli.errorhandler.ClientError: A client error (InvalidAccessKeyId) occurred when calling the ListObjects operation: The AWS Access Key Id you provided does not exist in our records. 2015-08-06 20:54:27,877 - awscli.clidriver - DEBUG - Exiting with rc 255 A client error (InvalidAccessKeyId) occurred when calling the ListObjects operation: The AWS Access Key Id you provided does not exist in our records.
```
上面的错误信息非常有误导性的一句话是:
* A client error (InvalidAccessKeyId) occurred when calling the ListObjects operation: The AWS Access Key Id you provided does not exist in our records.
* 然后你打电话给 support(记住一定要提供request id), 那边给的答复是你本机的时间不对
* WTF! 服务器肯定开启了NTP, 怎么会时间不对!
* 其实你使用 aws s3 --region cn-north-1 ls s3://your-bucket 就不会出错. 或者在 ~/.aws/config 中 配置:
* [default]region = cn-north-1
* 但是:
* support为什么会说我的时间不对?
* 为什么 aws client 报错是 The AWS Access Key Id you provided does not exist in our records
* 因为你的请求到了AWS 的美国区(或者准确说是非cn-north-1区)!
* 简单猜测一下原因(纯猜测, 猜对了才奇怪!):
* AWS S3 要高可用, 必须要存储多份数据, 而中国区只有一个availability zone(现在已经有多个了), 因此数据必须存储到其他region, 也就是说在内部, AWS cn-north-1 去跟其他region网络是通的!
* 默认情况下aws s3 的endpoint url 是其他region. 因此那个ls 操作直接请求了非cn-north-1 region.
* 但是aws cn-north-1 的账户系统跟其他region不通, 因此美国区返回错误: The AWS Access Key Id you provided does not exist in our records
* support 之所以根据request id 告诉你错误是因为请求时间不对, 也很简单: server端验证了请求的发起时间, 由于时差, 导致时间肯定是非法的. 因此support 告诉你说你的时间有问题
* 感觉客户端跟support告诉你的错误不一致是吧? 我当时就是因为他们的误导, 折腾了2天啊!!! 最后加一行代码解决了问题, 想死的❤️都有
* 因此结论很简单:
* 使用awscli 操作 S3 时, 记得带上 --region cn-north-1
* 写代码访问S3 时, 显示调用 setEndpoint 设置api地址
* // 关键是下面这一行, 在除了中国外的其他region, 这行代码不用写s3.setEndpoint(serviceEndpoint);
* S3 一个理解错误的坑
* S3 是一个KV 存储, 不存储在文件夹的概念. 比如你存储数据到 s3://yourbucket/a/b/c/d.txt, S3 仅仅是将s3://yourbucket/a/b/c/d.txt 作为key, value就是文件的内容. 如果你想ls s3://yourbucket/a/b 是不存在的key!
* S3 定位错误的tips
* 调试模式下, 可以考虑关闭ssl, 并使用 tcpdump 抓包查看数据是否正确, 非常实用
* aws 客户端可以添加 --debug 开启调试日志, 出错后开case时最好带着 Request ID 和 Extended Request ID . AWS 几乎所有服务的每次请求都是带有 Request ID 的, 非常便于定位问题. 至于为什么, 建议看Google早年的论文: Dapper, a Large-Scale Distributed Systems Tracing Infrastructure
* 聊完了 S3, 其他的基本上坑不多, 走过路过也记不得了. 但最深刻的一个关于 IAM 的要注意.
* Amazon IAM 坑
* 啥是IAM?
* AWS Identity and Access Management (IAM) 使您能够安全地控制用户对 Amazon AWS 服务和资源的访问权限。您可以使用 IAM 创建和管理 AWS 用户和群组，并使用各种权限来允许或拒绝他们对 AWS 资源的访问。
* 唯一大坑: IAM policy file 中 arn 的写法
* 啥是arn?
* Amazon Resource Names (ARNs) uniquely identify AWS resources. We require an ARN when you need to specify a resource unambiguously across all of AWS, such as in IAM policies, Amazon Relational Database Service (Amazon RDS) tags, and API calls.
* 具体参加这里
* 简单来说, arn 就是AWS中资源的uri. 任何AWS资源都可以用 arn 标识, 因此对于资源的访问控制配置文件也要使用 arn 来写.
* arn 的格式如下:
* arn:partition:service:region:account:resourcearn:partition:service:region:account:resourcetype/resourcearn:partition:service:region:account:resourcetype:resource
* 上面这行代码据说 在AWS 其他region 都可以使用
* 唯独中国区不能用! 因为AWS 中国区非常特殊, 上述文件中的 aws 要修改成 aws-cn !!!!
* 这样写在中国区就可以用:
* {  "Version": "2012-10-17",  "Statement": [    {      "Effect": "Allow",      "Action": "s3:*",      "Resource": ["arn:aws:s3:::your-bucket", "arn:aws:s3:::your-bucket/*"]    } ]}
* 不要小看这一点小区别, 由于AWS 其他region 都是用 aws 就可以, 因此很多开源项目中, 将 arn:aws: XXXX hard code 在代码里, 导致这些项目用到中国区会失败!
* BTW, 一个小坑: 上面的配置文件中的 "Version": "2012-10-17", 这个日期是必须写成这个的, 估计是AWS 的码农 hard code 的版本, 不能修改其他任何值 , 千万别用这个值来作为自己的版本控制(偷笑)
* 建议程序访问AWS 资源时, 使用IAM role的方式, 不要使用配置文件中写入AccessKey/Secret 的方式, 非常不安全.
* EC2
* EC2 就是虚拟主机. AWS 有两个概念: Reserved Instance 和 Spot Instance
* Reserved Instance
* 简单来说就是包年购买节点. 优点肯定是便宜. 缺点就是买了就不能退货. 但这里最坑(不容易理解)的是:
* 购买N个T类型的RI后, 其实仅仅是在RI有效期限内计费的时候, 该类型的节点中的N 个 T 类型节点按照打折价格计费.
* 即使你在RI 期限内没有使用任何该类型的 EC2 节点, 费用照常收取, RI 过期后价格恢复原价
* 其他节点已久按照正常价格按小时收费
* RI 仅仅是计费单元, 节点销毁后重新启动, 只要不超过 RI 数量, 都按照打折计费
* 例如: 购买了3个 t2.micro 类型的RI, 但是你再次期间最多同时开启了5个 t2.micro 节点, 那么这其中的3个是按照打折价格计费, 2个节点按照正常价格. 如果发现三台 t2.micro 配置错误, 直接 terminate 后启动新的instance , 依旧是按照 RI 的价格计费
* Spot Instance
* 这个就是可以以非常便宜的价格买到 EC2 节点. 不过迄今未知(2015-08-07) 中国区没有该项业务.