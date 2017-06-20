---
title: Hbase集群搭建(亲测)
date: 2017-06-15 14:17:27
categories:	hbase
tags: 
	- hbase
---

<!-- toc -->



#### Hbase集群搭建注意点
	Hbase端口 60010 
	1.上传hbase安装包
	
	2.解压
	
	3.配置hbase集群，要修改3个文件（首先zk集群已经安装好了）
		注意：要把hadoop的hdfs-site.xml和core-site.xml 放到hbase/conf下
		补充：还需要将hdfs-site.xml core-site.xml 拷贝到HRegionServer所在服务器上
		3.1修改hbase-env.sh
		export JAVA_HOME=/usr/java/jdk1.7.0_55
		//告诉hbase使用外部的zk
		export HBASE_MANAGES_ZK=false
		
		vim hbase-site.xml
		<configuration>
			<!-- 指定hbase在HDFS上存储的路径 -->
			<property>
					<name>hbase.rootdir</name>
					<value>hdfs://ns1/hbase</value>
			</property>
			<!-- 指定hbase是分布式的 -->
			<property>
					<name>hbase.cluster.distributed</name>
					<value>true</value>
			</property>
			<!-- 指定zk的地址，多个用“,”分割 -->
			<property>
					<name>hbase.zookeeper.quorum</name>
					<value>itcast04:2181,itcast05:2181,itcast06:2181</value>
			</property>
		</configuration>
		
		vim regionservers
		itcast03
		itcast04
		itcast05
		itcast06
		
		3.2拷贝hbase到其他节点
			scp -r /itcast/hbase-0.96.2-hadoop2/ itcast02:/itcast/
			scp -r /itcast/hbase-0.96.2-hadoop2/ itcast03:/itcast/
			scp -r /itcast/hbase-0.96.2-hadoop2/ itcast04:/itcast/
			scp -r /itcast/hbase-0.96.2-hadoop2/ itcast05:/itcast/
			scp -r /itcast/hbase-0.96.2-hadoop2/ itcast06:/itcast/
	4.将配置好的HBase拷贝到每一个节点并同步时间。
	
	5.启动所有的hbase
		分别启动zk
			./zkServer.sh start
		启动hbase集群
			start-dfs.sh
		启动hbase，在主节点上运行：
			start-hbase.sh
	6.通过浏览器访问hbase管理页面            
		192.168.1.201:60010   (好像端口是16010)
	7.为保证集群的可靠性，要启动多个HMaster
     hbase-daemon.sh start master