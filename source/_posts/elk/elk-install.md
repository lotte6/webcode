---
title: ELK 集群安装
categories: elk
tag: hide
date: 2020-4-4 16:50:49
tags:
---

比较简单官方提供的yum安装，或者apt安装，一样的思路，都是添加官方源，然后安装。

如果对docker比较熟悉，还可以使用docker安装

cat >/etc/yum.repos.d/es.repo<<eof
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
eof
yum install elasticsearch kibana

cat >/etc/elasticsearch/elasticsearch.yml<<eof
cluster.name: cn-log
node.name: ${HOSTNAME}
node.master: true
node.data: true
node.ingest: false
node.ml: false
node.max_local_storage_nodes: 1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: _site_
http.port: 9200
http.cors.enabled: true
http.cors.allow-origin: "*"
transport.tcp.port: 9300
discovery.zen.ping.unicast.hosts: ["172.20.3.17","172.20.3.18","172.20.3.19"]
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping_timeout: "5s"
action.destructive_requires_name: true
action.auto_create_index: true
indices.recovery.max_bytes_per_sec: "1024m"
cluster.routing.allocation.node_concurrent_incoming_recoveries: 30
cluster.routing.allocation.node_concurrent_outgoing_recoveries: 30
cluster.routing.allocation.node_initial_primaries_recoveries: 30
cluster.routing.allocation.cluster_concurrent_rebalance: 30
indices.queries.cache.size: 10%
indices.fielddata.cache.size: 20%
indices.breaker.fielddata.limit: 60%
indices.breaker.request.limit: 40%
indices.breaker.total.limit: 70%
xpack.ml.enabled: false
xpack.security.enabled: false
xpack.watcher.enabled: false
xpack.monitoring.exporters.my_local:
  type: local
  use_ingest: false
thread_pool.index.queue_size: 7000
thread_pool.bulk.queue_size: 7000
reindex.remote.whitelist: ["10.2.*.*:9200", "172.19.*.*:9200", "172.120.*.*:9200"]
path.repo: ["/esbackup"]
eof

cat >/etc/kibana/kibana.yml<<eof
server.host: "0.0.0.0"
elasticsearch.url: "http://172.20.3.19:9200"
eof


systemctl start kibana elasticsearch