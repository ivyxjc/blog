---
author: ivyxjc
date: 2020-04-07
title:  安装Kubernetes
category: 效率
tags: [kubernetes]
keywords:
description: 如何安装Kubernetes
toc: true
---

本文介绍在Ubuntu 18.04中如何利用kubeadm安装kubernetes


## 安装相应的软件

```shell
添加源（阿里云镜像）
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
echo 'deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main' >  /etc/apt/sources.list.d/kubernetes.list 
apt update
apt install -y kubelet kubeadm kubectl kubernetes-cni
```

## 启动kubernetes master


### 准备阶段

因为国内无法连接gcr.io，所以需要使用阿里云镜像将对应的docker images先pull下来

```shell
# 从aliyun中将对应的images pull下来, image name 是类似 registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.0  ...
kubeadm config images list |sed -e 's/^/docker pull /g' -e 's#k8s.gcr.io#registry.aliyuncs.com/google_containers#g' |sh -x
# 复制上面pull的image，并将将image name改为 k8s.gcr.io/kube-apiserver:v1.18.0 ...
docker images |grep google_containers |awk '{print "docker tag ",$1":"$2,$1":"$2}' |sed -e 's#registry.aliyuncs.com/google_containers#k8s.gcr.io#2' |sh -x
# 删除名字 类似registry.aliyuncs.com/google_containers 这些image
docker images |grep google_containers |awk '{print "docker rmi ", $1":"$2}' |sh -x
```

### init master

```bash
kubeadm init
```
在init完毕后，命令行会出现一些提示，按照提示操作


```bash
To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

下面这个就是下一章节 配置网络的提示
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

### 配置网络

此时执行`kubectl get nodes`会看到master的状态是`Not Ready`，这是因为CNI网络插件没有安装好。使用Calico插件

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

可以执行`kubectl get pods --all-namespace`来查看相关的Pod是否都正常启动。可以执行`kubectl --namespace=kube-system describe pod <pod_name>`来看错误原因。

### 错误恢复

可以执行`kubeadm reset`来撤销`kubeadm init`所造成的影响。 但是要注意命令行的提示，有些影响需要手动reset。

## Dashboard

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml`

然后启动proxy， 执行`kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$'`

nginx需要做对应的配置。

访问url
`https://k8sdashboard.ivyxjc.com/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/` 即可访问

1.0的版本因为namespace是`kube-system`，所以url是`https://k8sdashboard.ivyxjc.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`


### 安装遇到的问题

安装1.10.0的时候遇到了一些问题。和这篇文章[https://www.pomit.cn/p/2412183455255041](Web基础配置篇（十七）: Kubernetes dashboard安装配置)遇到的问题是一样的

## Cheat Sheet

```shell

kubectl delete pod -n kubernetes-dashboard kubernetes-dashboard-5d4dc8b976-4rv9h 

kubectl delete deployment -n kubernetes-dashboard kubernetes-dashboard

kubectl --namespace=kube-system describe pod kubernetes-dashboard-975499656-7swl4

kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$'

```

## 参考文章
1. [https://www.pomit.cn/p/2412183455255041](Web基础配置篇（十七）: Kubernetes dashboard安装配置) ([https://web.archive.org/web/20200407092613/https://www.pomit.cn/p/2412183455255041/](Internet Archive))