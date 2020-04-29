---
title: Index lifecycle management (ILM)
categories: elk
tag: hide
date: 2020-4-4 16:40:49
tags:
---

ElasticStack从2019年1月29日的6.6.0版本的开始，引入了索引生命周期管理的功能，新版本的Filebeat则默认的配置开启了ILM，导致索引的命名规则被ILM策略控制。


修改配置文件：
```
# filebeat 配置关闭 ILM 即可解决Index Pattern不生效的问题
setup.ilm.enabled: false
# 如果要开启ILM的话，这里是Filebeat ILM相关配置的文档
# https://www.elastic.co/guide/en/beats/filebeat/current/ilm.html

```
ElasticSearch作为一个全文搜索引擎，索引即是灵魂。在海量数据的场景下，管理索引是非常有挑战的事情：

增长过快的索引通常需要切分成多个提高搜索效率；
时效性强的数据，比如日志和监控数据，需要定期清理或者归档；
旧数据通过索引压缩，减少分片节约计算资源；
数据冷热分离到SSD和HDD硬盘节约存储成本等等
曾经这些事情需要专业的ES运维，人工处理或者借助Curator这样的开源工具来操作。而现在新版本的ES自带的索引生命周期管理（ILM），是解决这些问题的最佳实践的官方标准。ILM把索引分成4个阶段，每个阶段能够调整索引的优先级，在达到临界条件后执行相应的Action：

Hot阶段 热数据，索引优先级可以比较高，增长过快可以Rollover
Warm阶段 还有余热的数据，可以合并索引，减少Shard分片（Shrink），设置只读等等，
Cold阶段 冷数据，食之无味弃之可惜，这些数据可能已经很久都不会查询到一次，但是可能哪天还会用到，可以当做归档数据Freeze掉了，
Delete阶段 这些数据可能一辈子都不会查询到了，那就快刀斩乱麻，给新数据腾点存储空间吧
这四个阶段，每个阶段都有独特的Action，也有多个阶段都可以使用的Action，比如Allocate就可以在Warm和Cold两个阶段使用，用来灵活的改变分片Shard数量，副本Replicas数量以及存储节点，比如这个ILM Policy就可以实现数据的冷热分离：

```
# 存储两个副本, 并且转移索引数据到冷数据节点
PUT _ilm/policy/log_data_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "allocate" : {
            "number_of_replicas": 2,
            "require" : {
              "box_type": "cold"
            }
        }
        }
      }
    }
  }
}
```
为什么这个ILM策略可以实现冷热分离呢? 我们知道ES集群Master节点用来协调调度, Client节点用来处理API请求, Data节点用来索引存储数据。其中Data节点可以配置 node.attr.box_type，比如SSD节点设置这个值为hot，HDD节点设置这个值为cold，那么对于达到Warm条件的索引数据, 就会转移到HDD节点了，也就实现了冷热分离。关于不同阶段不同Action的策略配置，详见下面的文档链接：

https://www.elastic.co/guide/en/elasticsearch/reference/current/_actions.html

ILM的简单实践
在学习应用ElasticStack时，看官方文档是个很棒的办法，文档非常详细，此处Mark一下传送门：

https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

以及ES索引生命周期管理的文档传送门：

https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html

我们可以一边参考文档, 一边在Kibana的Dev Tools里面编写API请求来试验, Dev Tools堪称神器, 光标移到每一个请求上Ctrl + Enter立刻见效:
```
# 常用的 _cluster API 6连
GET _cluster/health?pretty

GET _cluster/health?pretty&level=indices

GET _cluster/health?pretty&level=indices&level=shards

GET _cluster/settings

GET _cluster/state

GET _cluster/stats?human&pretty

# 常用的 _cat API 3连
GET _cat/nodes?v

GET _cat/templates?v

GET _cat/indices?v

# 常用的 _node API 3连
GET _nodes?pretty

GET _nodes/stats

GET _nodes/usage
```
刀磨好了，开始砍柴。给索引添加ILM策略有两个方式，一是调用直接Create/Update Index的API设置单个索引的生命周期管理策略，二是通过Index Template关联ILM策略，这样可以一劳永逸，所有同一个Template的Indices都能被ILM控制。Filebeat MetricBeat等全家桶组件也都是用这种方式的。我们来尝试把现有的一个Index Template设置ILM。

第一步, 创建ILM策略, 比如这样的策略: 每天或者达到50GB轮滚一次, 30天后缩成1个分片,合并索引,并且增加副本, 60天后转移到冷数据节点, 90天后删除:
```
PUT _ilm/policy/log_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_size": "50G"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "number_of_shards": 1
          },
          "allocate": {
            "number_of_replicas": 2
          }
        }
      },
      "cold": {
        "min_age": "60d",
        "actions": {
          "allocate": {
            "require": {
              "box_type": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```
第二步 找到Template, 通过 GET _template/log-xxx 查看一下setting, 确认没有 settings.index.lifecycle.name/rollover_alias 这两个属性
第三步 更新这个Index Template, 应用第一步创建的 “log_policy”
```
PUT _template/log-xxx
{
  "index_patterns": ["log-xxx-*"], 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index.lifecycle.name": "log_policy", 
    "index.lifecycle.rollover_alias": "log-xxx"
  }
}
```
第四步 查看Policy以及Template创建出来的新的索引策略是否应用成功
GET _ilm/policy

GET log-xxx-*/_ilm/explain
# managed 为 true 即已经被ILM管理
只要几个API就能完成这么复杂的索引生命周期管理!

与Curator对比
Curator是一个Python实现的开源的ES索引管理项目: https://github.com/elastic/curator

在ILM出现之前，Curator可以说是一个ElasticSearch索引管理的必备品，定时创建新的索引，删除、合并、压缩、备份/恢复旧的索引，通过简单的YAML配置 + Crontab即可实现。那么有了ILM之后，Curator的定位就比较尴尬了，Curator能做的ILM都能做，而且还更简单。官方文档也给出了最佳实践，翻译概括一下就是：咱的Beats系列和Logstash都能利用ILM的特性，还不赶紧丢掉Curator来用ILM？
https://www.elastic.co/guide/en/elasticsearch/client/curator/5.7/ilm-or-curator.html


附一个实例

```
PUT /_ilm/policy/deletepolicy
{
    "policy": {
        "phases": {
            "hot": {
                "actions": {
                    "rollover": {
                        "max_age": "14d",
                        "max_size": "50gb"
                    }
                }
            },
            "warm": {
                "min_age": "3d",
                "actions": {
                    "allocate": {
                        "require": {
                            "box_type": "warm"
                        },
                        "number_of_replicas": 0
                    },
                    "forcemerge": {
                        "max_num_segments": 1
                    },
                    "shrink": {
                        "number_of_shards": 1
                    }
                }
            },
            "cold": {
                "min_age": "7d",
                "actions": {
                    "allocate": {
                        "require": {
                            "box_type": "cold"
                        }
                    }
                }
            },
            "delete": {
                "min_age": "30d",
                "actions": {
                    "delete": {}
                }
            }
        }
    }
}
}}
```

