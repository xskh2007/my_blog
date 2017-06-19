---
title: zabbix3.2.1分布式监控，采用agent主动模式
date: 2017-06-15 14:17:27
categories:	zabbix
tags: 
	- zabbix
---

<!-- toc -->



# zabbix3.2.1分布式监控，采用agent主动模式

有点：1、分布式，全部agent先提交到各个机房的zabbix_proxy，再有zabbix_proxy提交到到server，减少					zabbix_server压力
   


服务端 安装参考  Zabbix 3.2.0 yum或编译安装 on CentOS 7.note
配置制动注册让agent自动注册，免去一个个手动添加

proxy 安装参考 zabbix分布式监控proxy部署.note



cd /root/zabbix-3.2.1
./configure --prefix=/usr/local/zabbix_agent --enable-agent 
 make install
修改 vim /usr/local/zabbix_agent/etc/zabbix_agentd.conf
#Server 写proxy的ip
Server=172.16.1.198 

useradd zabbix
/usr/local/zabbix_agent/sbin/zabbix_agentd -c /usr/local/zabbix_agent/etc/zabbix_agentd.conf

错误1 没发现host，如果是被动模式的话就是就是server上没配对应的host，但是我用的是主动模式，不需要手动添加，只要添加自动加host的action就行，我遇到的问题是这个action没启动，我记得是启用了得。不知道什么时候被禁用了！！！
[图片]
解决方法：
[图片]



测试方法：
[root@xs_proxy ~]# /usr/local/zabbix/bin/zabbix_get -s 172.17.2.201 -p20050 -k"system.uptime"
16936953