---
title: kubectl 常用命令
categories: k8s
tag: hide
date: 2020-04-3 16:37:49
tags:
---

kubectl命令---
获取namespace信息：
kubectl get namespace
查看node详细信息：
kubectl describe node
kubectl get node

帮助信息--
kubectl scale -h
查看版本--
kubectl version
查看所有的pod---
kubectl get pods --all-namespaces     查看所有pod；
kubectl get deployment --all-namespaces   查看部署的应用；
kubectl get deployment --namespace=kube-system kong -o yaml
kubectl delete deployment --namespace=kube-system kong  删除指定的应用；
kubectl get services
kubectl get pod ded98392a-9d72-442f-905-3544635521-m02mv --namespace=d7c25b925-8ba7-44c3-b95
kubectl get pod ded98392a-9d72-442f-905-3544635521-m02mv --namespace=d7c25b925-8ba7-44c3-b95（查看指定namespace的pod详细描述信息）

按照yaml或json格式查看pod信息：
kubectl get pod d2616548f-ea69-4244-819-1317460255-5d6pa --namespace=df432da7a-3a36-490f-a7b -o yaml
kubectl get endpoints
kubectl get namespaces
应用namespace是随着项目走，项目不同，namespace就不同；
kubectl命令将deployment替换RC，不再使用RC命令；
查看k8s的manage的pod的镜像的版本号：
kubectl get pod --namespace=kube-system               manage-2792156093-ijz4l -o yaml
查看指定的service：
kubectl get service kubernetes
查看node主机：
kubectl get node --no-headers -L zone
查看service应用：
kubectl get endpoints --all-namespaces
kubectl describe service d5c4e0793-5b44-4b89-a51 --namespace=d07a12d56-fe42-4b27-a18
查看端点endpoint:
kubectl get endpoints --all-namespaces
查看replicaset：
replicaset是用来管理实例数量的，可以看成是rc/deployment的一个对象。
kubectl get replicaset --all-namespaces
删除指定replicaset:
 kubectl delete replicaset --namespace=d4c71895a-c152-4c03-ba5   de31d32c8-9205-4b8e-b12-2472488937
查看节点数据：
kubectl describe/get node 10.158.99.12

查看节点资源：
kubectl get resourcequota --all-namespaces -o yaml
查看镜像：
sudo docker images
sudo docker images | grep kong
查看指定类型pod：
kubectl get pod --all-namespaces -o yaml | grep manage
查看log：
kubectl logs -f --namespace=kube-system               manage-3943165616-l8ahz
查看node信息：
kubectl get node -a
kubectl get node --show-labels
kubectl get node --show-all
查看endpoints信息---->
容器虚ip/内部端口号---查看
 kubectl get endpoints --namespace=d07a12d56-fe42-4b27-a18
--------------------- 



cp etcd etcdctl /k8s/etcd/bin/

rm -fr /var/lib/etcd/*

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd server-csr.json | cfssljson -bare server
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server


cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server


./kube-apiserver \
--logtostderr=true \
--v=4 \
--etcd-servers=https://172.16.29.37:2379,https://172.16.29.38:2379 \
--bind-address=0.0.0.0 \
--secure-port=6443 \
--advertise-address=172.16.8.37 \
--allow-privileged=true \
--service-cluster-ip-range=10.0.0.0/24 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth \
--token-auth-file=/k8s/kubernetes/cfg/token.csv \
--service-node-port-range=30000-50000 \
--tls-cert-file=/k8s/kubernetes/ssl/server.pem \
--tls-private-key-file=/k8s/kubernetes/ssl/server-key.pem \
--client-ca-file=/k8s/kubernetes/ssl/ca.pem \
--service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem \
--etcd-cafile=/k8s/etcd/ssl/ca.pem \
--etcd-certfile=/k8s/etcd/ssl/server.pem \
--etcd-keyfile=/k8s/etcd/ssl/server-key.pem

./etcdctl --endpoints=https://172.16.29.38:2379,https://172.16.29.37:2379 --cert-file=/k8s/etcd/ssl/server.pem --ca-file=/k8s/etcd/ssl/ca.pem --key-file=/k8s/etcd/ssl/server-key.pem member list

kubectl get cs,nodes 不指定证书可能报错

kubectl certificate approve node-csr-j9QqwRtTGNS9Dc6eotc2QGw3uejJCsgH79aK1nTAnx0

etcdctl \
--ca-file=/k8s/etcd/ssl/ca.pem \
--cert-file=/k8s/etcd/ssl/server.pem \
--key-file=/k8s/etcd/ssl/server-key.pem \
--endpoints="https://172.16.29.37:2379, \
https://172.16.29.38" cluster-health


etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://172.16.29.37:2379,https://172.16.29.38" cluster-health

systemctl restart etcd
systemctl restart kube-apiserver
systemctl restart kube-scheduler.service
systemctl restart kube-controller-manager

systemctl restart kubelet
systemctl restart kube-proxy
kubectl get csr

kubectl create clusterrolebinding kubelet-nodes --clusterrole=system:node --group=system:nodes

systemctl start etcd
systemctl start kube-apiserver
systemctl start kube-scheduler.service
systemctl start kube-controller-manager

systemctl start kubelet
systemctl start kube-proxy


systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy


systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet

systemctl stop etcd
systemctl stop kube-apiserver
systemctl stop kube-scheduler.service
systemctl stop kube-controller-manager
stop
systemctl stop kubelet
systemctl stop kube-proxy


```
cat << EOF | tee server-csr.json
{
"CN": "etcd",
"hosts": [
"172.16.29.37",
"172.16.29.38"
],
"key": {
"algo": "rsa",
"size": 2048
},
"names": [
{
"C": "CN",
"L": "Beijing",
"ST": "Beijing"
}
]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd server-csr.json | cfssljson -bare server
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server


cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server



cat << EOF | tee server-csr.json
{
"CN": "kubernetes",
"hosts": [
"10.254.0.1",
"127.0.0.1",
"172.16.29.37",
"172.16.29.38",
"kubernetes",
"kubernetes.default",
"kubernetes.default.svc",
"kubernetes.default.svc.cluster",
"kubernetes.default.svc.cluster.local"
],
"key": {
"algo": "rsa",
"size": 2048
},
"names": [
{
"C": "CN",
"L": "Beijing",
"ST": "Beijing",
"O": "k8s",
"OU": "System"
}
]
}
EOF


cfssl gencert -initca ca-csr.json | cfssljson -bare ca -



Name: admin-user-token-9rtds
Namespace: kube-system
Labels: <none>
Annotations: kubernetes.io/service-account.name: admin-user
kubernetes.io/service-account.uid: 8213fd0b-3b25-11e9-a51f-005056a12a57

Type: kubernetes.io/service-account-token

Data
====
ca.crt: 1359 bytes
namespace: 11 bytes
token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTlydGRzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4MjEzZmQwYi0zYjI1LTExZTktYTUxZi0wMDUwNTZhMTJhNTciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.cfAsc1nGn5bBY3PW6KS_IFwO7Lz4xRlSzpEQiNZRbC8RXx-lR45naFvS5BXg6jaRgvUymVYeNU65610KHuuVa6jJwHfk6hoLEuCaZ2ubSKKnqER1W7LgGd1EfLpI4mBZxhzQrlIW6_xDsvsIGWmRelvhZvvg6DH7HRjo48MB1dMhP0y-ncbkF1y4RhCEKLtYuMR4wVIH3qNSU0wkHXEZgTzzeWk6b0bsXX4AG4P-OlrCMr9z0JC3m76wubcA8nliZHspaIf6bnvI6SmgPepI1Io3TaHL812ExkanDiF0bSIo-LaoKI375vLPQt_h6tbfRTXlX2rqiOh1A62wSfpkHQ
```

kubectl get pods --all-namespaces

kubectl get all --all-namespaces
kubectl get secret,sa,role,rolebinding,services,deployments --namespace=kube-system | grep dashboard
kubectl create clusterrolebinding system:anonymous --clusterrole=cluster-admin --user=system:anonymous
kubectl cluster-info

kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml

kubectl delete deployment kubernetes-dashboard --namespace=kube-system 
kubectl delete service kubernetes-dashboard --namespace=kube-system 
kubectl delete role kubernetes-dashboard-minimal --namespace=kube-system 
kubectl delete rolebinding kubernetes-dashboard-minimal --namespace=kube-system
kubectl delete sa kubernetes-dashboard --namespace=kube-system 
kubectl delete secret kubernetes-dashboard-certs --namespace=kube-system
kubectl delete secret kubernetes-dashboard-key-holder --namespace=kube-system


kubectl edit deployment kubernetes-dashboard --namespace kube-system


kubectl -n kube-system get service kubernetes-dashboard

kubectl -n kube-system edit service kubernetes-dashboard
kubectl get svc -n kube-system

--basic-auth-file=/k8s/kubernetes/basic_auth_file \
--anonymous-auth=false

kubectl get svc,deployment,pod -n kube-system
kubectl get pods --all-namespaces -o wide

kubectl get all --all-namespaces -o wide
kubectl get svc,deployment,pod -n kube-system


/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://172.16.29.37:2379,https://172.16.29.38:2379" set /k8s/network/config '{ "Network": "10.254.0.0/16", "Backend": {"Type": "vxlan"}}'



http://traefik/dashboard



KUBE_PROXY_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=172.16.29.66 \
--cluster-cidr=10.254.0.0/16 \
#--proxy-mode=ipvs \
--kubeconfig=/k8s/kubernetes/cfg/kube-proxy.kubeconfig"




kubectl get pods --all-namespaces     查看所有pod；
kubectl get deployment --all-namespaces   查看部署的应用；
kubectl get deployment --namespace=kube-system kong -o yaml
kubectl delete deployment --namespace=kube-system kong  删除指定的应用；
kubectl get services
kubectl get pod ded98392a-9d72-442f-905-3544635521-m02mv --namespace=d7c25b925-8ba7-44c3-b95
kubectl get pod ded98392a-9d72-442f-905-3544635521-m02mv --namespace=d7c25b925-8ba7-44c3-b95（查看指定namespace的pod详细描述信息）

按照yaml或json格式查看pod信息：
kubectl get pod d2616548f-ea69-4244-819-1317460255-5d6pa --namespace=df432da7a-3a36-490f-a7b -o yaml
kubectl get endpoints
kubectl get namespaces
应用namespace是随着项目走，项目不同，namespace就不同；
kubectl命令将deployment替换RC，不再使用RC命令；
查看k8s的manage的pod的镜像的版本号：
kubectl get pod --namespace=kube-system               manage-2792156093-ijz4l -o yaml
查看指定的service：
kubectl get service kubernetes
查看node主机：
kubectl get node --no-headers -L zone
查看service应用：
kubectl get endpoints --all-namespaces
kubectl describe service d5c4e0793-5b44-4b89-a51 --namespace=d07a12d56-fe42-4b27-a18
查看端点endpoint:
kubectl get endpoints --all-namespaces
查看replicaset：
replicaset是用来管理实例数量的，可以看成是rc/deployment的一个对象。
kubectl get replicaset --all-namespaces
删除指定replicaset:
 kubectl delete replicaset --namespace=d4c71895a-c152-4c03-ba5   de31d32c8-9205-4b8e-b12-2472488937
查看节点数据：
kubectl describe/get node 10.158.99.12

查看节点资源：
kubectl get resourcequota --all-namespaces -o yaml
查看镜像：
sudo docker images
sudo docker images | grep kong

