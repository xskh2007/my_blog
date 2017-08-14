---
title: 基于kubernetes集群部署DashBoard二(亲测)
date: 2017-08-14 10:38:16
categories:	k8s
tags: 
	- k8s
---

在之前一篇文章：Centos7部署Kubernetes集群，中已经搭建了基本的K8s集群，本文将在此基础之上继续搭建K8s DashBoard。
## yaml文件
编辑dashboard.yaml，注意或更改以下部分：
gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
-  --apiserver-host=http://10.0.251.148:8080

	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	# Keep the name in sync with image version and
	# gce/coreos/kube-manifests/addons/dashboard counterparts
	  name: kubernetes-dashboard-latest
	  namespace: kube-system
	spec:
	  replicas: 1
	  template:
		metadata:
		  labels:
			k8s-app: kubernetes-dashboard
			version: latest
			kubernetes.io/cluster-service: "true"
		spec:
		  containers:
		  - name: kubernetes-dashboard
			image: gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
			resources:
			  # keep request = limit to keep this container in guaranteed class
			  limits:
				cpu: 100m
				memory: 50Mi
			  requests:
				cpu: 100m
				memory: 50Mi
			ports:
			- containerPort: 9090
			args:
			 -  --apiserver-host=http://10.0.251.148:8080
			livenessProbe:
			  httpGet:
				path: /
				port: 9090
			  initialDelaySeconds: 30
			  timeoutSeconds: 30
			  
编辑dashboardsvc.yaml文件：

	apiVersion: v1
	kind: Service
	metadata:
	  name: kubernetes-dashboard
	  namespace: kube-system
	  labels:
		k8s-app: kubernetes-dashboard
		kubernetes.io/cluster-service: "true"
	spec:
	  selector:
		k8s-app: kubernetes-dashboard
	  ports:
	  - port: 80
		targetPort: 9090
		
## 镜像准备
在dashboard.yaml中定义了dashboard所用的镜像：gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1（当然你可以选择其他的版
本），另外，启动k8s的pod还需要一个额外的镜像：registry.access.redhat.com/rhel7/pod-infrastructure:latest（node中，/etc/kubernetes/kubele
t的配置），由于一些众所周知的原因，这两个镜像在国内是下载不下来的，以下介绍如何准备这两个镜像。

### 从阿里云下载
阿里云docker下载地址：https://dev.aliyun.com/search.html
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/K8S/aliyun1.png) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/K8S/aliyun1.png) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/K8S/aliyun1.png) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/K8S/aliyun1.png) 
从阿里云上pull下来对应的镜像，之后通过docker save保存成tar包，在每个node上执行docker load将镜像导入。类似的命令如下：

	各个node上执行：
	docker pull registry.cn-hangzhou.aliyuncs.com/kube_containers/kubernetes-dashboard-amd64
	docker save registry.cn-hangzhou.aliyuncs.com/kube_containers/kubernetes-dashboard-amd64 > dashboard.tar
	docker pull registry.cn-hangzhou.aliyuncs.com/architect/pod-infrastructure
	docker save registry.cn-hangzhou.aliyuncs.com/architect/pod-infrastructure > podinfrastructure.tar

	docker load < dashboard.tar
	docker load < podinfrastructure.tar

### 搭梯子(plan B)
　　在node所在同网段（相同交换机）内，搭建一个可以正常访问google、Facebook等网站的fq网关，
将集群中所有机器的GATEWAY指向该地址，之后重启网络。这样，所有的机器就能够正常下载这两个镜像了。
	
## 启动
　　在master执行如下命令：
kubectl create -f dashboard.yaml
kubectl create -f dashboardsvc.yaml
　　之后，dashboard搭建完成。

## 验证
命令验证，master上执行如下命令：

	[root@k8s-master ~]# kubectl get deployment --all-namespaces
	NAMESPACE NAME DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	kube-system kubernetes-dashboard-latest 1  1         1            1           1h
	[root@k8s-master ~]# kubectl get svc  --all-namespaces
	NAMESPACE   NAME          CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
	default       kubernetes             10.254.0.1      <none>        443/TCP   9d
	kube-system   kubernetes-dashboard   10.254.44.119   <none>        80/TCP    1h
	[root@k8s-master ~]# kubectl get pod  -o wide  --all-namespaces
	NAMESPACE  NAME  READY     STATUS    RESTARTS     AGE   IP  NODE
	kube-system kubernetes-dashboard-latest-3866786896-vsf3h 1/1 Running 0 1h 10.0.82.2 k8s-node-1

界面验证，浏览器访问：http://192.168.4.155:8080/ui
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/K8S/dsb.jpg) 