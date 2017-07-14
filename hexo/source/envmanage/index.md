---
title: envmanage
categories:	sql审核
tags: #envmanage
	- envmanage
date: 2017-07-12 10:46:32
---


<!-- toc -->



## 项目开发背景 ##

	1、由于多项目并行开发，系统架构不断拆分导致一个环境就需要几十台虚拟机。需要很多的开发测试环境。
	2、一个项目上线后这些环境又会有没人维护的尴尬局面。继续给后面的项目用的话又会有各种缺少表结构，
	   配置文件。包依赖等问题。
	3、现阶段如果新加一个环境的话基本上没2.3天时间是搞不定的。而且会有各种遗漏和手误。有些错误甚至是很难发现的
#### 新建一个环境的具体事情如下：####
	
	1、批量克隆虚拟机(基于zstack企业版)
	2、批量创建对应的jenkinsjob
	3、跳板机资产等级
	4、nginx配置文件修改为对应的ip
	5、虚拟机的host文件，dns文件。修改
	等等
		

## 项目功能程流程图 ##

![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/envmanage/envmanage.png) 

## 项目的开发方法
	在jumpserver0.3.2基础上加celery异步任务实现
	注意jumpserver0.3.2只支持python2.6，而新版本的celery只支持python2.7以上
	所以在安装celery时需要指定版本
	具体如下:
	pip install celery==3.1.20
	pip install django-celery==3.1.17
 
## 项目地址
	Git 地址  https://github.com/xskh2007/zjump 
	QQ群：391096485

## 项目功能附图
	待开发。。。