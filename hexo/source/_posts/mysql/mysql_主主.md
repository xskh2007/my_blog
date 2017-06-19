---
title: mysql主主同步配置
date: 2017-06-15 14:17:27
categories:	mysql
tags: 
	- mysql
---

<!-- toc -->




#### 准备工作

	setenforce 0  
	service iptables stop
	yum -y install mysql-server mysql-devel mysql
	service mysqld start
	service mysqld stop


#### 192.168.128.140上操作

	[root@localhost ~]# vim /etc/my.cnf 

	[client]
	default-character-set=utf8
	[mysqld]
	wait_timeout         = 864000
	interactive_timeout  = 864000
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	user=mysql
	# Disabling symbolic-links is recommended to prevent assorted security risks
	symbolic-links=0

	auto_increment_offset=1
	auto-increment-increment=2
	server_id = 140
	log-bin=mysql-bin
	character_set_server=utf8
	replicate_wild_ignore_table=information_schema.%
	replicate-wild_ignore_table=performance_schema.%
	replicate-wild_ignore_table=test.%

	[mysqld_safe]
	log-error=/var/log/mysqld.log
	pid-file=/var/run/mysqld/mysqld.pid

	[root@localhost ~]# service mysqld restart




#### 192.168.128.137 上操作

[root@localhost ~]# vi /etc/my.cnf 

	[client]
	default-character-set=utf8
	[mysqld]
	wait_timeout         = 864000
	interactive_timeout  = 864000
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	user=mysql
	# Disabling symbolic-links is recommended to prevent assorted security risks
	symbolic-links=0
	
	auto_increment_offset=2
	auto-increment-increment=2
	server_id = 053
	log-bin=mysql-bin
	character_set_server=utf8
	replicate_wild_ignore_table=information_schema.%
	replicate-wild_ignore_table=performance_schema.%
	replicate-wild_ignore_table=test.%
	
	[mysqld_safe]
	log-error=/var/log/mysqld.log
	pid-file=/var/run/mysqld/mysqld.pid
	
	[root@localhost ~]# service mysqld restart
	


#### 192.168.128.140 上操作

	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.128.137' IDENTIFIED BY 'twlqccr';
	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.128.140' IDENTIFIED BY 'twlqccr';
	mysql> flush tables with read lock; 
	mysql> show master status\G
	*************************** 1. row ***************************
				File: mysql-bin.000001
			Position: 430
		Binlog_Do_DB:
	Binlog_Ignore_DB:
	1 row in set (0.00 sec)
	
	导出数据，scp到192.168.128..137
	mysqldump qiantu >/tmp/qiantu.sql 
	scp /tmp/qiantu.sql 192.168.128.137:/tmp/
	
	mysql>UNLOCK TABLES;
	
	
	192.168.128.137 上操作
	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.128.137' IDENTIFIED BY 'twlqccr';
	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.128.140' IDENTIFIED BY 'twlqccr';
	mysql> create database qiantu ;
	mysql> use qiantu;
	mysql> source /tmp/qiantu.sql;
	
	stop slave;
	reset slave;
	change master to master_host='192.168.128.140',master_user='repl', master_password='twlqccr', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos=430;
	start slave;
	show slave status\G
	
#### 在192.168.128.137看到两个yes就代表192.168.128.140(主)192.168.128.137(从)成功了


反过来再做一次，就是主主了

#### 192.168.128.137 查看

	mysql> flush tables with read lock; 
	mysql> show master status\G
	*************************** 1. row ***************************
				File: mysql-bin.000001
			Position: 1500
		Binlog_Do_DB:
	Binlog_Ignore_DB:
	1 row in set (0.00 sec)
	
	
	
	192.168.128.140 上操作
	stop slave;
	reset slave;
	change master to master_host='192.168.128.137',master_user='repl', master_password='twlqccr', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos=1500;
	start slave;
	show slave status\G

#### 在192.168.128.140看到两个yes就代表主从192.168.128.137(主)192.168.128.140(从)成功了


测试，新建一个数据库 去另一台机器看看是否有生成




#### 测试脚本

	#!/bin/bash
	mysql -e "create database qiantu;"
	mysql -e "create table qiantu.demo(addtime timestamp);"
	while true
	do
	mysql -e "select * from qiantu.demo;"
	sleep 3
	mysql -e "insert into qiantu.demo values(null);"
	done





#如果是克隆的第二台机器，要改下uuid，不然会报错
vim /var/lib/mysql/auto.cnf
[auto]
server-uuid=99424caf-4807-11e6-a88d-005056806ac1

