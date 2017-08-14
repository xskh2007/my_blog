---
title: 批量克隆jenkins的job
date: 2017-07-31 16:08:22
categories:	jenkins
tags: 
	- jenkins
---

<!-- toc -->


## 添加一个job

### 创建一个mavn项目
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/create_job.jpg) 
### 修改job配置如图：
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/1.jpg) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/2.jpg) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/3.jpg) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/4.jpg) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/5.jpg) 
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/6.jpg) 

## 找到需要克隆的job配置文件

	cd /data/jenkins_data/jobs/xiasha_AAA_cms

	[root@jenkins xiasha_AAA_cms]# ll
	total 20
	drwxr-x--- 13 root root 4096 Jul 24 19:40 builds
	-rw-r-----  1 root root 5876 Jul 24 19:38 config.xml
	lrwxrwxrwx  1 root root   22 Jul 24 19:39 lastStable -> builds/lastStableBuild
	lrwxrwxrwx  1 root root   26 Jul 24 19:39 lastSuccessful -> builds/lastSuccessfulBuild
	drwxr-x---  7 root root 4096 Jul 21 11:53 modules
	-rw-r-----  1 root root    3 Jul 24 19:39 nextBuildNumber
	[root@jenkins xiasha_AAA_cms]# vim config.xml 
	[root@jenkins xiasha_AAA_cms]# pwd
	/data/jenkins_data/jobs/xiasha_AAA_cms
	[root@jenkins xiasha_AAA_cms]# cat config.xml
	<?xml version='1.0' encoding='UTF-8'?>
	<maven2-moduleset plugin="maven-plugin@2.17">
	  <actions/>
	  <description></description>
	  <keepDependencies>false</keepDependencies>
	  <properties>
		<hudson.model.ParametersDefinitionProperty>
		  <parameterDefinitions>
			<hudson.model.StringParameterDefinition>
			  <name>model</name>
			  <description></description>
			  <defaultValue>cms</defaultValue>
			</hudson.model.StringParameterDefinition>
			<net.uaznia.lukanus.hudson.plugins.gitparameter.GitParameterDefinition plugin="git-parameter@0.8.0">
			  <name>branch</name>
			  <description></description>
			  <uuid>9d451b9b-9c6b-46ae-93be-636c522bac6f</uuid>
			  <type>PT_BRANCH_TAG</type>
			  <branch></branch>
			  <tagFilter>*</tagFilter>
			  <branchFilter>.*</branchFilter>
			  <sortMode>NONE</sortMode>
			  <defaultValue></defaultValue>
			  <selectedValue>NONE</selectedValue>
			  <quickFilterEnabled>false</quickFilterEnabled>
			</net.uaznia.lukanus.hudson.plugins.gitparameter.GitParameterDefinition>
		  </parameterDefinitions>
		</hudson.model.ParametersDefinitionProperty>
	  </properties>
	  <scm class="hudson.plugins.git.GitSCM" plugin="git@2.5.3">
		<configVersion>2</configVersion>
		<userRemoteConfigs>
		  <hudson.plugins.git.UserRemoteConfig>
			<url>git@gitlab.lark.wiki:nbgold/${model}.git</url>
			<credentialsId>e646bdce-28d6-4a65-a169-f1a916517b24</credentialsId>
		  </hudson.plugins.git.UserRemoteConfig>
		</userRemoteConfigs>
		<branches>
		  <hudson.plugins.git.BranchSpec>
			<name>$branch</name>
		  </hudson.plugins.git.BranchSpec>
		</branches>
		<doGenerateSubmoduleConfigurations>false</doGenerateSubmoduleConfigurations>
		<submoduleCfg class="list"/>
		<extensions/>
	  </scm>
	  <canRoam>true</canRoam>
	  <disabled>false</disabled>
	  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
	  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
	  <jdk>(System)</jdk>
	  <triggers/>
	  <concurrentBuild>false</concurrentBuild>
	  <rootModule>
		<groupId>com.zzjr</groupId>
		<artifactId>cms</artifactId>
	  </rootModule>
	  <goals>clean install -Dmaven.test.skip=true</goals>
	  <aggregatorStyleBuild>true</aggregatorStyleBuild>
	  <incrementalBuild>false</incrementalBuild>
	  <ignoreUpstremChanges>false</ignoreUpstremChanges>
	  <ignoreUnsuccessfulUpstreams>false</ignoreUnsuccessfulUpstreams>
	  <archivingDisabled>false</archivingDisabled>
	  <siteArchivingDisabled>false</siteArchivingDisabled>
	  <fingerprintingDisabled>false</fingerprintingDisabled>
	  <resolveDependencies>false</resolveDependencies>
	  <processPlugins>false</processPlugins>
	  <mavenValidationLevel>-1</mavenValidationLevel>
	  <runHeadless>false</runHeadless>
	  <disableTriggerDownstreamProjects>false</disableTriggerDownstreamProjects>
	  <blockTriggerWhenBuilding>true</blockTriggerWhenBuilding>
	  <settings class="jenkins.mvn.DefaultSettingsProvider"/>
	  <globalSettings class="jenkins.mvn.DefaultGlobalSettingsProvider"/>
	  <reporters/>
	  <publishers/>
	  <buildWrappers/>
	  <prebuilders/>
	  <postbuilders>
		<jenkins.plugins.publish__over__ssh.BapSshBuilderPlugin plugin="publish-over-ssh@1.14">
		  <delegate>
			<consolePrefix>SSH: </consolePrefix>
			<delegate>
			  <publishers>
				<jenkins.plugins.publish__over__ssh.BapSshPublisher>
				  <configName>ansible</configName>
				  <verbose>false</verbose>
				  <transfers>
					<jenkins.plugins.publish__over__ssh.BapSshTransfer>
					  <remoteDirectory></remoteDirectory>
					  <sourceFiles>${model}-webapp/target/*.war</sourceFiles>
					  <excludes></excludes>
					  <removePrefix></removePrefix>
					  <remoteDirectorySDF>false</remoteDirectorySDF>
					  <flatten>true</flatten>
					  <cleanRemote>false</cleanRemote>
					  <noDefaultExcludes>false</noDefaultExcludes>
					  <makeEmptyDirs>false</makeEmptyDirs>
					  <patternSeparator>[, ]+</patternSeparator>
					  <execCommand>ansible ${model}_xsy_01 -m shell -a &quot;mkdir -p /data/webapps/&quot; --sudo 
	ansible ${model}_xsy_01 -m shell -a &quot;echo &apos;zkserver=xxl01.niubangold.com:2181,xxl02.niubangold.com:2181,xxl03.niubangold.com:2181&apos;&gt;/data/webapps/xxl-conf.properties&quot; --sudo 
	ansible ${model}_xsy_01 -m shell -a &quot;service tomcat stop&quot; --sudo
	ansible ${model}_xsy_01 -m shell -a &quot;rm -rf /usr/local/tomcat/webapps/*&quot; --sudo

	ansible ${model}_xsy_01 -m copy -a &quot;src=/usr/local/war/${model}.war dest=/usr/local/tomcat/webapps/ROOT.war&quot; --sudo

	ansible ${model}_xsy_01 -m shell -a &quot;service tomcat start&quot; --sudo</execCommand>
					  <execTimeout>120000</execTimeout>
					  <usePty>false</usePty>
					</jenkins.plugins.publish__over__ssh.BapSshTransfer>
				  </transfers>
				  <useWorkspaceInPromotion>false</useWorkspaceInPromotion>
				  <usePromotionTimestamp>false</usePromotionTimestamp>
				</jenkins.plugins.publish__over__ssh.BapSshPublisher>
			  </publishers>
			  <continueOnError>false</continueOnError>
			  <failOnError>false</failOnError>
			  <alwaysPublishFromMaster>false</alwaysPublishFromMaster>
			  <hostConfigurationAccess class="jenkins.plugins.publish_over_ssh.BapSshPublisherPlugin" reference="../.."/>
			</delegate>
		  </delegate>
		</jenkins.plugins.publish__over__ssh.BapSshBuilderPlugin>
	  </postbuilders>
	  <runPostStepsIfResult>
		<name>FAILURE</name>
		<ordinal>2</ordinal>
		<color>RED</color>
		<completeBuild>true</completeBuild>
	  </runPostStepsIfResult>
	</maven2-moduleset>
	
## 新建脚本

	#!/usr/bin/env python
	# -*- coding:utf-8 -*- 
	#Author: qiantu
	#qq 261767353

	import jenkins
	giturl="git@gitlab.lark.wiki:nbgold/"
	template_xml="jenkins_job.xml"
	view_name = "xiasha_BBB_"
	server = jenkins.Jenkins('http://192.168.2.169:8080/jenkins/', username='admin', password='zzjr#2015')

	import xml.etree.cElementTree as ET
	job_list=["cms","pay","member","weapon","trade","wealth","statistics","bops","napp","site","crm","channel","tiger","rams","activity","api","eagle","squirrel","wap","dragon","tengu"]
	tree = ET.ElementTree(file=template_xml)

	def create_job_file(model):
		for ele in tree.iterfind("properties/hudson.model.ParametersDefinitionProperty/parameterDefinitions/hudson.model.StringParameterDefinition/defaultValue"):
			ele.text=model
			ele.set("updated","up")
		for ele in tree.iterfind("scm/userRemoteConfigs/hudson.plugins.git.UserRemoteConfig/url"):
			ele.text=giturl+model+".git"
			ele.set("updated","up")
		file_name="jenkins_job_"+model+".xml"
		tree.write(file_name)
		return open(file_name).read()

	for i in job_list:
		EMPTY_JOB_CONFIG=create_job_file(i)
		# print EMPTY_JOB_CONFIG

		new_job = server.create_job(view_name+i, EMPTY_JOB_CONFIG)
		# del_job=server.delete_job(view_name+i)
		
## 运行结果：
![Screenshot](https://raw.githubusercontent.com/xskh2007/xskh2007.github.io/master/images/python/jieguo.jpg) 