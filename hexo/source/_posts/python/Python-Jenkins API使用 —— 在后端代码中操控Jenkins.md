---
title: Python-Jenkins API使用 —— 在后端代码中操控Jenkins
date: 2017-06-15 14:17:27
categories:	jenkins
tags: 
	- jenkins
---

<!-- toc -->





#### Python-Jenkins API使用 —— 在后端代码中操控Jenkins

最近在工作中需要用到在后台代码中触发Jenkins任务的构建，于是想到Jenkins是否有一些已经封装好的API类库提供，用于处理跟Jenkins相关的操作。下面就简单介绍下我的发现。

#### Linux Curl

首先找到的是Jenkins官网的wiki：https://wiki.jenkins-ci.org/display/JENKINS/Remote+access+API

在官网首页就有关于触发job的方法：



个人尝试了下，该方式是通过命令行直接调curl去发POST请求的方式来触发job的构建。对于用openid管理的Jenkins，需要带上参数--user USER:PASSWORD，其中的USER和PASSWORD不是你的openID登录的账号密码，而是登录后显示在Jenkins中的User Id和API Token，它们的的查看方式如下：

用openID登录jenkins —> 点击右上角的用户名，进入用户个人页面 —>  点击左边的设置，打开设置页面 —> API Token，Show Api Token...

如果需要参数化构建job，则要加上--data-urlencode json='{"parameter": [{"name":"param_name1","value":"param_value1"}, {"name":"param_name2","value":"param_value2"}]}'

显然，这种方式比较繁琐，很容易出现因格式不正确导致触发任务失败，而且这种方式不能帮助我们获取更多的关于job的信息以便于我们后续对job的状态进行跟踪。

#### Python-Jenkins

继续寻找，然后我在Jenkins官网上找到了Python-Jenkins API，仔细阅读后发现，它几乎涵盖了大部分Jenkins的操作，大大方便了我们在后台进行对Jenkins的一些列操作。

Python-Jenkins官网：https://pypi.python.org/pypi/python-jenkins/

Python-Jenkins Doc：http://python-jenkins.readthedocs.io/en/latest/index.html

下面简单介绍下如何使用Python-Jenkins：

#### 1. 安装

	sudo pip install python-jenkins

#### 2. 进入python命令环境或创建新的.py文件jenkinsApiTest.py

	import jenkins
	#定义远程的jenkins master server的url，以及port
	jenkins_server_url='xxxx:xxxx'
	#定义用户的User Id 和 API Token，获取方式同上文
	user_id='xxxx'
	api_token='xxxx'
	#实例化jenkins对象，连接远程的jenkins master server
	server=jenkins.Jenkins(jenkins_server_url, username=user_id, password=api_token)
	#构建job名为job_name的job（不带构建参数）
	server.build_job(job_name)
	#String参数化构建job名为job_name的job, 参数param_dict为字典形式，如：param_dict= {"param1"：“value1”， “param2”：“value2”}
	server.build_job(job_name, parameters=param_dict)
	#获取job名为job_name的job的相关信息
	server.get_job_info(job_name)
	#获取job名为job_name的job的最后次构建号
	server.get_job_info(job_name)['lastBuild']['number']
	#获取job名为job_name的job的某次构建的执行结果状态
	server.get_build_info(job_name,build_number)['result']　　
	#判断job名为job_name的job的某次构建是否还在构建中
	server.get_build_info(job_name,build_number)['building']

　　3. 更多其他的API可以参考Python-Jenkins API：http://python-jenkins.readthedocs.io/en/latest/api.html
