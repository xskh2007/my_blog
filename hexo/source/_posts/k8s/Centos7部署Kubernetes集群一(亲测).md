---
title: Centos7部署Kubernetes集群一(亲测)
date: 2017-08-14 10:38:16
categories:	k8s
tags: 
	- k8s
---

## 环境介绍及准备：
### 操作系统(物理机,虚拟机都可以)

操作系统采用Centos7.3 64位，细节如下。

	[root@k8s-master ~]# uname -a
	Linux k8s-master 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
	[root@k8s-master ~]# cat /etc/redhat-release 
	CentOS Linux release 7.3.1611 (Core) 


### 主机信息

	本文准备了三台机器用于部署k8s的运行环境，细节如下：
	
节点及功能 | 主机名 | IP
---|---|---
Master、etcd、registry | K8s-master| 192.168.4.155
Node1 | K8s-node-1| 192.168.4.156
Node2 | K8s-node-2| 192.168.4.157


设置三台机器的主机名：
Master上执行：
	[root@localhost ~]#  hostnamectl --static set-hostname  k8s-master	
Node1上执行：
	[root@localhost ~]# hostnamectl --static set-hostname  k8s-node-1	
Node2上执行：
	[root@localhost ~]# hostnamectl --static set-hostname  k8s-node-2
	
在三台机器上设置hosts，均执行如下命令：

	echo '192.168.4.155    k8s-master
	192.168.4.155   etcd
	192.168.4.155   registry
	192.168.4.156   k8s-node-1
	192.168.4.157    k8s-node-2' >> /etc/hosts
	
### 关闭三台机器上的防火墙
	systemctl disable firewalld.service
	systemctl stop firewalld.service
	
## 部署etcd
k8s运行依赖etcd，需要先部署etcd，本文采用yum方式安装：

	[root@localhost ~]# yum install etcd -y

yum安装的etcd默认配置文件在/etc/etcd/etcd.conf。编辑配置文件，更改以下信息：
ETCD_NAME=master
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd:2379,http://etcd:4001"

	[root@localhost ~]# vi /etc/etcd/etcd.conf
	# [member]
	ETCD_NAME=master
	ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
	#ETCD_WAL_DIR=""
	#ETCD_SNAPSHOT_COUNT="10000"
	#ETCD_HEARTBEAT_INTERVAL="100"
	#ETCD_ELECTION_TIMEOUT="1000"
	#ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
	ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"
	#ETCD_MAX_SNAPSHOTS="5"
	#ETCD_MAX_WALS="5"
	#ETCD_CORS=""
	#
	#[cluster]
	#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
	# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
	#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
	#ETCD_INITIAL_CLUSTER_STATE="new"
	#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
	ETCD_ADVERTISE_CLIENT_URLS="http://etcd:2379,http://etcd:4001"
	#ETCD_DISCOVERY=""
	#ETCD_DISCOVERY_SRV=""
	#ETCD_DISCOVERY_FALLBACK="proxy"
	#ETCD_DISCOVERY_PROXY=""

启动并验证状态

	[root@localhost ~]# systemctl start etcd
	[root@localhost ~]#  etcdctl set testdir/testkey0 0
	0
	[root@localhost ~]#  etcdctl get testdir/testkey0 
	0
	[root@localhost ~]# etcdctl -C http://etcd:4001 cluster-health
	member 8e9e05c52164694d is healthy: got healthy result from http://0.0.0.0:2379
	cluster is healthy
	[root@localhost ~]# etcdctl -C http://etcd:2379 cluster-health
	member 8e9e05c52164694d is healthy: got healthy result from http://0.0.0.0:2379
	cluster is healthy
	
扩展：Etcd集群部署参见——http://www.cnblogs.com/zhenyuyaodidiao/p/6237019.html

## 部署master
### 安装Docker
	
	[root@k8s-master ~]# yum install docker
	
配置Docker配置文件，使其允许从registry中拉取镜像。增加如下一行：
OPTIONS='--insecure-registry registry:5000'

	[root@k8s-master ~]# vim /etc/sysconfig/docker

	# /etc/sysconfig/docker

	# Modify these options if you want to change the way the docker daemon runs
	OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
	if [ -z "${DOCKER_CERT_PATH}" ]; then
		DOCKER_CERT_PATH=/etc/docker
	fi
	
	OPTIONS='--insecure-registry registry:5000'
	
设置开机自启动并开启服务

	[root@k8s-master ~]# chkconfig docker on
	[root@k8s-master ~]# service docker start
	
### 安装kubernets

	[root@k8s-master ~]# yum install kubernetes
	
### 配置并启动kubernetes
在kubernetes master上需要运行以下组件：
　　　　Kubernets API Server
　　　　Kubernets Controller Manager
　　　　Kubernets Scheduler
相应的要更改以下几个配置中带颜色部分信息：

#### 修改 /etc/kubernetes/apiserver 
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBE_ETCD_SERVERS="--etcd-servers=http://etcd:2379"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"

	[root@k8s-master ~]# vim /etc/kubernetes/apiserver

	###
	# kubernetes system config
	#
	# The following values are used to configure the kube-apiserver
	#

	# The address on the local server to listen to.
	KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"

	# The port on the local server to listen on.
	KUBE_API_PORT="--port=8080"

	# Port minions listen on
	# KUBELET_PORT="--kubelet-port=10250"

	# Comma separated list of nodes in the etcd cluster
	KUBE_ETCD_SERVERS="--etcd-servers=http://etcd:2379"

	# Address range to use for services
	KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

	# default admission control policies
	#KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
	KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"

	# Add your own!
	KUBE_API_ARGS=""
	
#### 修改/etc/kubernetes/config 
KUBE_MASTER="--master=http://k8s-master:8080"

	[root@k8s-master ~]# vim /etc/kubernetes/config

	###
	# kubernetes system config
	#
	# The following values are used to configure various aspects of all
	# kubernetes services, including
	#
	#   kube-apiserver.service
	#   kube-controller-manager.service
	#   kube-scheduler.service
	#   kubelet.service
	#   kube-proxy.service
	# logging to stderr means we get it in the systemd journal
	KUBE_LOGTOSTDERR="--logtostderr=true"

	# journal message level, 0 is debug
	KUBE_LOG_LEVEL="--v=0"

	# Should this cluster be allowed to run privileged docker containers
	KUBE_ALLOW_PRIV="--allow-privileged=false"

	# How the controller-manager, scheduler, and proxy find the apiserver
	KUBE_MASTER="--master=http://k8s-master:8080"
	
#### 启动服务并设置开机自启动

	[root@k8s-master ~]# systemctl enable kube-apiserver.service
	[root@k8s-master ~]# systemctl start kube-apiserver.service
	[root@k8s-master ~]# systemctl enable kube-controller-manager.service
	[root@k8s-master ~]# systemctl start kube-controller-manager.service
	[root@k8s-master ~]# systemctl enable kube-scheduler.service
	[root@k8s-master ~]# systemctl start kube-scheduler.service

## 部署node
### 安装docker
　　参见3.1
### 安装kubernets
　　参见3.2
### 配置并启动kubernetes
在kubernetes node上需要运行以下组件：
Kubelet
Kubernets Proxy
相应的要更改以下几个配置文中信息：
#### 修改 /etc/kubernetes/config
KUBE_MASTER="--master=http://k8s-master:8080"

	[root@K8s-node-1 ~]# vim /etc/kubernetes/config

	###
	# kubernetes system config
	#
	# The following values are used to configure various aspects of all
	# kubernetes services, including
	#
	#   kube-apiserver.service
	#   kube-controller-manager.service
	#   kube-scheduler.service
	#   kubelet.service
	#   kube-proxy.service
	# logging to stderr means we get it in the systemd journal
	KUBE_LOGTOSTDERR="--logtostderr=true"

	# journal message level, 0 is debug
	KUBE_LOG_LEVEL="--v=0"

	# Should this cluster be allowed to run privileged docker containers
	KUBE_ALLOW_PRIV="--allow-privileged=false"

	# How the controller-manager, scheduler, and proxy find the apiserver
	KUBE_MASTER="--master=http://k8s-master:8080"

#### 修改/etc/kubernetes/kubelet 
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_HOSTNAME="--hostname-override=k8s-node-1" (注意第二台要写 k8s-node-2)
KUBELET_API_SERVER="--api-servers=http://k8s-master:8080"

	[root@K8s-node-1 ~]# vim /etc/kubernetes/kubelet

	###
	# kubernetes kubelet (minion) config

	# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
	KUBELET_ADDRESS="--address=0.0.0.0"

	# The port for the info server to serve on
	# KUBELET_PORT="--port=10250"

	# You may leave this blank to use the actual hostname
	KUBELET_HOSTNAME="--hostname-override=k8s-node-1"

	# location of the api-server
	KUBELET_API_SERVER="--api-servers=http://k8s-master:8080"

	# pod infrastructure container
	KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"

	# Add your own!
	KUBELET_ARGS=""
	
启动服务并设置开机自启动

	[root@k8s-master ~]# systemctl enable kubelet.service
	[root@k8s-master ~]# systemctl start kubelet.service
	[root@k8s-master ~]# systemctl enable kube-proxy.service
	[root@k8s-master ~]# systemctl start kube-proxy.service
	
### 查看状态
在master上查看集群中节点及节点状态

	[root@k8s-master ~]#  kubectl -s http://k8s-master:8080 get node
	NAME         STATUS    AGE
	k8s-node-1   Ready     3m
	k8s-node-2   Ready     16s
	[root@k8s-master ~]# kubectl get nodes
	NAME         STATUS    AGE
	k8s-node-1   Ready     3m
	k8s-node-2   Ready     43s
至此，已经搭建了一个kubernetes集群，但目前该集群还不能很好的工作，请继续后续的步骤。

## 创建覆盖网络——Flannel
### 安装Flannel
在master、node上均执行如下命令，进行安装

	[root@k8s-master ~]# yum install flannel

### 配置Flannel
master、node上均编辑/etc/sysconfig/flanneld，修改红色部分

	[root@k8s-master ~]# vi /etc/sysconfig/flanneld

	# Flanneld configuration options

	# etcd url location.  Point this to the server where etcd runs
	FLANNEL_ETCD_ENDPOINTS="http://etcd:2379"

	# etcd config key.  This is the configuration key that flannel queries
	# For address range assignment
	FLANNEL_ETCD_PREFIX="/atomic.io/network"

	# Any additional options that you want to pass
	#FLANNEL_OPTIONS=""
	
### 配置etcd中关于flannel的key
Flannel使用Etcd进行配置，来保证多个Flannel实例之间的配置一致性，所以需要在etcd上进行如下配置：
（‘/atomic.io/network/config’这个key与上文/etc/sysconfig/flannel中的配置项FLANNEL_ETCD_PREFIX是相对应的，
错误的话启动就会出错）

	[root@k8s-master ~]# etcdctl mk /atomic.io/network/config '{ "Network": "10.0.0.0/16" }'
	{ "Network": "10.0.0.0/16" }
	
### 启动
启动Flannel之后，需要依次重启docker、kubernete。
在master执行：

	systemctl enable flanneld.service 
	systemctl start flanneld.service 
	service docker restart
	systemctl restart kube-apiserver.service
	systemctl restart kube-controller-manager.service
	systemctl restart kube-scheduler.service
	
在node上执行：

	systemctl enable flanneld.service 
	systemctl start flanneld.service 
	service docker restart
	systemctl restart kubelet.service
	systemctl restart kube-proxy.service