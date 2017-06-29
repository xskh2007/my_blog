---
title: ceph这款云存储技术怎么样？和swift、hdfs相比如何
date: 2017-06-15 14:17:27
categories:	分布式文件系统
tags: 
	- 分布式文件系统
---

<!-- toc -->



Ceph是一套高性能，易扩展的，无单点的分布式文件存储系统，基于Sage A. Weil的论文开发，主要提供以下三个存储服务：

- > 对象存储(Object Storage)，既可以通过使用Ceph的库，利用C, C++, Java, Python, PHP代码，也可以通过Restful网关以对象的形式访问或存储数据，兼容亚马逊的S3和OpenStack的Swift。
- > 块存储(Block Storage)，作为块设备像硬盘一样直接挂载。
- > 文件系统(File System) ，如同网络文件系统一样挂载，兼容POSIX接口。


Ceph的结构，对象存储由LIBRADOS和RADOSGW提供，块存储由RBD提供，文件系统由CEPH FS提供，而RADOSGW, RBD, CEPH FS均需要调用LIBRADOS的接口，而最终都是以对象的形式存储于RADOS里。

Ceph集群的节点有三种角色：

- > Monitor，监控集群的健康状况，向客户端发送最新的CRUSH map(含有当前网络的拓扑结构)
- > OSD，维护节点上的对象，响应客户端请求，与其他OSD节点同步
- > MDS，提供文件的Metadata，如果不使用CephFS可以不安装

Ceph是分布式的存储，它将文件分割后均匀随机地分散在各个节点上，Ceph采用了CRUSH算法来确定对象的存储位置，
只要有当前集群的拓扑结构，Ceph客户端就能直接计算出文件的存储位置，
直接跟OSD节点通信获取文件而不需要询问中心节点获得文件位置，这样就避免了单点风险。
更多Ceph架构方面的内容可以参看官方介绍：
ArchitectureCeph目前已经是一套比较成熟的存储系统了，
是OpenStack比较理想的存储后端，也可以作为Hadoop的存储后端，这就涉及到与Swift和HDFS的比较。

### Ceph与Swift
Ceph用C++编写而Swift用Python编写，性能上应当是Ceph占优。
但是与Ceph不同，Swift专注于对象存储，作为OpenStack组件之一经过大量生产实践的验证，
与OpenStack结合很好，目前不少人使用Ceph为OpenStack提供块存储，但仍旧使用Swift提供对象存储。
Swift的开发者曾写过文章对比Ceph和Swift: Ceph and Swift: Why we are not fighting.

### Ceph与HDFS
Ceph对比HDFS优势在于易扩展，无单点。HDFS是专门为Hadoop这样的云计算而生，
在离线批量处理大数据上有先天的优势，而Ceph是一个通用的实时存储系统。
虽然Hadoop可以利用Ceph作为存储后端（根据Ceph官方的教程死活整合不了，自己写了个简洁的步骤:http://www.kai-zhang.com/cloud-computing/Running-Hadoop-on-CEPH/），
但执行计算任务上性能还是略逊于HDFS（时间上慢30%左右 Haceph: Scalable Meta- data Management for Hadoop using Ceph）。





