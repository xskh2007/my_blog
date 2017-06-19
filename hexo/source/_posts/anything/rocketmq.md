---
title: RocketMQ-Console-Ng  how to install
date: 2017-06-15 14:17:27
categories:	rocketmq
tags: 
	- rocketmq
---

<!-- toc -->




# RocketMQ-Console-Ng  how to install

	git clone https://github.com/apache/incubator-rocketmq-externals.git
	cd incubator-rocketmq-externals
	cd rocketmq-console
	mvn clean package -Dmaven.test.skip=true
	java -jar -Drocketmq.namesrv.addr=192.168.1.24:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false rocketmq-console-ng-1.0.0.jar
	