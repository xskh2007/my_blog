---
title: mysql群集
date: 2017-06-15 14:17:27
categories:	mysql
tags: 
	- mysql
---

<!-- toc -->


QXtraDB是MySQL开源的一个分支，在数据操作层面上，和MYSQL基本一样。其集群与常用的MYSQL集群特性如下：


个人认为XtraDB最大的优点，解决了传统集群的两个问题，
         一是，数据同步的及时性，解决了MYSQL做分步式方案时的时间差问题。
         二是实现了多主，各节点可同时读写，这样一来的话，在需要对大并发，大数据量的频繁操作处理的情况下，达到数据库层面负载分流的目的，且实现方便。同时有效的最大化利用了硬件资源。

局限性
1.目前的复制仅仅支持InnoDB存储引擎。任何写入其他引擎的表，包括mysql.*表将不会复制。但是DDL语句会被复制的，因此创建用户将会被复制，但是insert into mysql.user…将不会被复制的。


2.DELETE操作不支持没有主键的表。没有主键的表在不同的节点顺序将不同，如果执行SELECT…LIMIT… 将出现不同的结果集。


3.在多主环境下LOCK/UNLOCK TABLES不支持。以及锁函数GET_LOCK(), RELEASE_LOCK()…


4.查询日志不能保存在表中。如果开启查询日志，只能保存到文件中。


5.允许最大的事务大小由wsrep_max_ws_rows和wsrep_max_ws_size定义。任何大型操作将被拒绝。如大型的LOAD DATA操作。


6.由于集群是乐观的并发控制，事务commit可能在该阶段中止。如果有两个事务向在集群中不同的节点向同一行写入并提交，失败的节点将中止。对于集群级别的中止，集群返回死锁错误代码(Error: 1213 SQLSTATE: 40001 (ER_LOCK_DEADLOCK)).


7.XA事务不支持，由于在提交上可能回滚。


8.整个集群的写入吞吐量是由最弱的节点限制，如果有一个节点变得缓慢，那么整个集群将是缓慢的。为了稳定的高性能要求，所有的节点应使用统一的硬件。


9.集群节点建议最少3个。


10.如果DDL语句有问题将破坏集群。


***********************      以下是配置步骤   *******************************
以下测试在 redhat6.4 64bit下实现【RPM方式安装】，采用三台机器，信息如下：
     db1  192.168.163.189
     db2  192.168.163.190
      db3  192.168.163.191
1、各节点配置和安装 2个 YUM 源
[EPEL]
name=EPEL
baseurl=http://mirror.neu.edu.cn/fedora/epel//6Server/x86_64/
enabled=1
gpgcheck=0
rpm -Uhv http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm

2、各节点删除系统自带的相关软件包
  rpm -e --nodeps  mysql-libs-5.1.66-2.el6_3.x86_64

3、各节点安装相关软件
yum install Percona-XtraDB-Cluster-server -y
正常情况下应该会安装以下相关软件包 
     Percona-XtraDB-Cluster-server-55
     Percona-Server-shared-51  
     Percona-XtraDB-Cluster-client-55
     Percona-XtraDB-Cluster-galera-2 
     Percona-XtraDB-Cluster-shared-55
    percona-xtrabackup
    perl-DBD-MySQL

4、各节点my.cnf最基本的配置
[mysqld]
basedir=/usr #可修改
datadir=/var/lib/mysql #可修改
wsrep_provider=/usr/lib64/libgalera_smm.so #需确认路径是否正常
wsrep_cluster_address=gcomm://  #首次运行不指定，等整个集群启动后，再修改，否则集群无法启动
wsrep_slave_threads=8
wsrep_sst_method=rsync
binlog_format=ROW
default_storage_engine=InnoDB
innodb_locks_unsafe_for_binlog=1
innodb_autoinc_lock_mode=2
wsrep_sst_auth=wsuser:redhat   #同步时使用的用户名及密码，格式为用户名：密码

5、集群中第一个节点192.168.163.189首次启动，启动时留意数据库错误日志信息
/etc/init.d/mysql  bootstrap-pxc


6、第二个节点192.168.163.190首次启动
/etc/init.d/mysql start wsrep_cluster_address=gcomm://192.168.163.189,192.168.163.190

7 、第三个节点192.168.163.190首次启动 
/etc/init.d/mysql start wsrep_cluster_address=gcomm://192.168.163.189,192.168.163.190,192.168.163.191

【注】以下各节点在启动前后，可用命令查看集群信息情况
mySQL>show status like 'wsrep%';

8、各节点授权
授权：
MYSQL>CREATE USER 'wsuser'@'localhost' IDENTIFIED BY 'redhat';
 GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'wsuser'@'localhost';
 FLUSH PRIVILEGES;

当所有节点启动后，修改各节点的MY.CNF修改参数wsrep_cluster_address=gcomm://192.168.163.189,192.168.163.190,192.168.163.191

9、测试集群可用性
    1、在其中任意一个节点创建一个INNodb库，到其它节点检查是否已有库生成。