---
title: K8s 服务不间断滚动升级
categories: k8s
tag: hide
date: 2020-4-3 16:32:49
tags:
---

### 详解k8s零停机滚动发布微服务 - kubernetes

1、前言
在当下微服务架构盛行的时代，用户希望应用程序时时刻刻都是可用，为了满足不断变化的新业务，需要不断升级更新应用程序，有时可能需要频繁的发布版本。实现"零停机"、“零感知”的持续集成(Continuous Integration)和持续交付/部署(Continuous Delivery)应用程序，一直都是软件升级换代不得不面对的一个难题和痛点，也是一种追求的理想方式，也是DevOps诞生的目的。

2、滚动发布
把一次完整的发布过程，合理地分成多个批次，每次发布一个批次，成功后，再发布下一个批次，最终完成所有批次的发布。在整个滚动过程期间，保证始终有可用的副本在运行，从而平滑的发布新版本，实现零停机(without an outage)、用户零感知，是一种非常主流的发布方式。由于其自动化程度比较高，通常需要复杂的发布工具支撑，而k8s可以完美的胜任这个任务。

3、k8s滚动更新机制
k8s创建副本应用程序的最佳方法就是部署(Deployment)，部署自动创建副本集(ReplicaSet)，副本集可以精确地控制每次替换的Pod数量，从而可以很好的实现滚动更新。具体来说，k8s每次使用一个新的副本控制器(replication controller)来替换已存在的副本控制器，从而始终使用一个新的Pod模板来替换旧的pod模板。

大致步骤如下：

创建一个新的replication controller。
增加或减少pod副本数量，直到满足当前批次期望的数量。
删除旧的replication controller。
4、演示
使用kubectl更新一个已部署的应用程序，并模拟回滚。为了方便分析，将应用程序的pod副本数量设置为10。

kubectl -n k8s-ecoysystem-apps scale deployment helloworldapi  --replicas=10
4.1. 发布微服务
查看部署列表
$ kubectl get deployments -n k8s-ecoysystem-apps
查看正在运行的pod
$ kubectl get pods -n k8s-ecoysystem-apps
通过pod描述，查看应用程序的当前映像版本
$ kubectl describe pods -n k8s-ecoysystem-apps


升级镜像版本到v2.3
$ kubectl -n k8s-ecoysystem-apps set image deployments/helloworldapi helloworldapi=registry.wuling.com/justmine/helloworldapi:v2.3


4.2. 验证发布
检查rollout状态
kubectl -n k8s-ecoysystem-apps rollout status deployments/helloworldapi 
检查pod详情
kubectl describe pods -n k8s-ecoysystem-apps

从上图可以看到，镜像已经升级到v2.3版本

4.3. 回滚发布
kubectl -n k8s-ecoysystem-apps rollout undo deployments/helloworldapi 

到目前为止，整个滚动发布工作就圆满完成了！！！
那么如果我们想回滚到指定版本呢？答案是k8s完美支持，并且还可以通过资源文件进行配置保留的历史版次量。由于篇幅有限，感兴趣的朋友，可以自己下去实战，回滚命令如下：

kubectl -n k8s-ecoysystem-apps rollout undo deployment/helloworldapi  --to-revision=<版次>
5、原理
k8s精确地控制着整个发布过程，分批次有序地进行着滚动更新，直到把所有旧的副本全部更新到新版本。实际上，k8s是通过两个参数来精确地控制着每次滚动的pod数量：

maxSurge 滚动更新过程中运行操作期望副本数的最大pod数，可以为绝对数值(eg：5)，但不能为0；也可以为百分数(eg：10%)。默认为25%。
maxUnavailable 滚动更新过程中不可用的最大pod数，可以为绝对数值(eg：5)，但不能为0；也可以为百分数(eg：10%)。默认为25%。
如果未指定这两个可选参数，则k8s会使用默认配置：

kubectl -n k8s-ecoysystem-apps get deployment helloworldapi -o yaml


5.1. 剖析部署概况


DESIRED 最终期望处于READY状态的副本数
CURRENT 当前的副本总数
UP-TO-DATE 当前完成更新的副本数
AVAILABLE 当前可用的副本数
当前的副本总数 = 10 + 10 * 25% = 13，所以CURRENT为13。
当前可用的副本数 = 10 - 10 * 25% = 8，所以AVAILABLE为8。

5.2. 剖析部署详情
kubectl -n k8s-ecoysystem-apps describe deployment helloworldapi  


整个滚动过程是通过控制两个副本集来完成的，新的副本集：helloworldapi-6564f59f66；旧的副本集：helloworldapi-6f4959c8c7 。
理想状态下的滚动过程：

创建了一个新的副本集，并为其分配3个新版本的pod，使副本总数达到13，一切正常。
通知旧副本集，销毁2个旧版本的pod，使可用副本总数保持到8，一起正常。
当两个副本销毁成功后，通知新副本集，再新增2个新版本的pod，使副本总数达到13，一切正常。
只要销毁成功，新副本集就会创造新的pod，一直循环，直到旧的副本集pod数量为0。
滚动升级一个服务，实际就是创建一个新的RS，然后逐渐将新RS中副本数增加到理想状态，将旧RS中的副本数减小到0的复合操作；

无论理想还是不理想，k8s最终都会使应用程序全部更新到期望状态，都会始终保持最大的副本总数和可用副本总数的不变性！！！

6、总结
本篇详解了k8s滚动更新机制，并通过实战演示了微服务的滚动更新，当然还可以加入健康检查和历史版次回滚，大家可以下去自己实践，在实战中学习和进步，基础打牢后，我们将结合实际情况，实战更多的例子，下一篇将实战金丝雀发布微服务，请继续关注。

本篇已贡献给kubeasz，使用指南，特性实验，Rollingupdate

如果你觉得本篇文章对您有帮助的话，感谢您的【推荐】。
如果你对 kubernets 感兴趣的话可以关注我，我会定期的在博客分享我的学习心得。

7、延伸阅读
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rolling-update
https://kubernetes.io/docs/tutorials/kubernetes-basics/update-intro/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rolling-update
https://github.com/kubernetes/community/blob/master/contributors/design-proposals/cli/simple-rolling-update.md
https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller
https://kubernetes.io/docs/tutorials/kubernetes-basics/update-interactive
https://kubernetes.io/images/docs/kubectl_rollingupdate.svg


更新镜像
kubectl set image deployment/nginx-test nginx=nginx:1.15 
rancher  kubectl set image deployment/nginx-deployment nginx=nginx:1.14.2

kubectl edit deployment/nginx-deployment

回退到上一版本
kubectl rollout undo deployment/nginx-test



回退到指定版本



kubectl rollout undo deployment/nginx-test --to-revision=2


查看回滚状态
kubectl rollout status daemonset/foo

查看历史记录
kubectl rollout history deployment/nginx-test

kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi

kubectl rollout undo deployment/abc

查看配置
kubectl describe deployment nginx-deployment

隔离node
kubectl cordon node1
恢复
kubectl uncordon node1


暂停和继续
$ kubectl rollout pause deployment/nginx-deployment2
$ kubectl rollout resume deployment/nginx-deployment2
