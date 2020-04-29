---
title: ELK 删除所有数据
categories: elk
tag: hide
date: 2020-4-4 16:46:49
tags:
---

之前在 2.X版本里 这个Delete By Query功能被去掉了，因为官方认为会引发一些错误，如需使用 ，需要自己安装插件。

bin/plugin install delete-by-query
DELETE /索引名/需要清空的type/_query
    {
        "query": {
        "match_all": {}
    }
}

POST indexName/_delete_by_query
{
  "query": { 
    "match_all": {
    }
  }
}


有时候因为数据量较大删除到一半就返回结果了，不要慌，继续发送命令，多执行几次就好了。
之后删除索引
DELETE /indexName


#清空索引

POST quality_control/my_type/_delete_by_query?refresh&slices=5&pretty
{
  "query": {
    "match_all": {}
  }
}

DELETE /_all
DELETE /*


POST /logs_2014-01-*/_flush 
POST /logs_2014-01-*/_close 
POST /logs_2014-01-*/_open 


数据过期
https://www.elastic.co/guide/cn/elasticsearch/guide/current/retiring-data.html