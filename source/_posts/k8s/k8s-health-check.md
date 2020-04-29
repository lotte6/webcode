---
title: K8s health check
categories: k8s
tag: hide
date: 2020-03-16 16:37:49
tags:
---

##### kubernetes探针有许多配置项，您可以使用它们来更精确地控制如何进行就绪和生存检查：
* initialDelaySeconds：容器启动和探针启动之间的秒数。
* periodSeconds：检查的频率（以秒为单位）。默认为10秒。最小值为1。
* timeoutSeconds：检查超时的秒数。默认为1秒。最小值为1。
* successThreshold：失败后检查成功的最小连续成功次数。默认为1.活跃度必须为1。最小值为1。
* failureThreshold：当Pod成功启动且检查失败时，Kubernetes将在放弃之前尝试failureThreshold次。放弃生存检查意味着重新启动* * Pod。而放弃就绪检查，Pod将被标记为未就绪。默认为3.最小值为1。
* HTTP探针在httpGet上的配置项：
* host：主机名，默认为pod的IP。
* scheme：用于连接主机的方案（HTTP或HTTPS）。默认为HTTP。
* path：探针的路径。
* httpHeaders：在HTTP请求中设置的自定义标头。 HTTP允许重复的请求头。
* port：端口的名称或编号。数字必须在1到65535的范围内。

##### liveness 的配置来自 v1中的Probe资源，所有属性含义如下：
* httpGet：对应HTTPGetAction对象，属性包括：host、httpHeaders、path、port、scheme
* initialDelaySeconds：容器启动后开始探测之前需要等多少秒，如应用启动一般30s的话，就设置为 30s
* periodSeconds：执行探测的频率（多少秒执行一次）。默认为10秒。最小值为1。
* successThreshold：探针失败后，最少连续成功多少次才视为成功。默认值为1。最小值为1。
* failureThreshold：最少连续多少次失败才视为失败。默认值为3。最小值为1。
* timeoutSeconds：探测的超时时间，默认 1s，最小 1s
* tcpSocket：对应TCPSocketAction对象，TCPSocket指定端口。尚不支持TCP hook
* exec：对应ExecAction对象，需要执行的内容

