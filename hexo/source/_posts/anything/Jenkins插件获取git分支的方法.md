---
title: Jenkins插件获取git分支的方法
date: 2017-07-14 16:10:27
categories:	jenkins
tags: 
	- jenkins
---

<!-- toc -->




## Jenkins插件获取git分支的方法

环境：Centos6.5_x86_64

Jenkins:1.65和2.46.2

 

    公司内部的测试环境中使用的Jenkins环境是1.65，现在已经更新了很多个版本了，但是由于一直正常使用也未升级；其实只要功能、安全、稳定性可以一般都很少经常升级的；但是为了跟上开源的步伐在虚拟机上做了一个新版本的测试；发现之前的好几个插件都已经在新版中去掉了；本次仅记录经常使用到的git代码分支获取的插件问题；

    jenkins可以通过参数化构建，可以极大方便了开发部署，各种参数传入方便后续调用，使用shell脚本或Python进行处理。

 

## 1、旧版本的Jenkins可以使用Dynamic Choice Parameter插件；

使用方法：

Jenkins--->dev-h5-server--->配置--->参数化构建过程--->选择Dynamic Choice Parameter插件：

    Name:   git_branch 
     
    Choices Script ： 
    def gettags = ("git ls-remote -h http://10.0.10.25/h5-server.git").execute() 
    gettags.text.readLines().collect { it.split()[1].replaceAll('refs/heads/', '')  }.unique() 

源码管理--->Git--->	Branches to build

    把*/master 改成：$git_branch [就是上面定义的Name值] 

这样就可以获取到git代码分支了；

 

我在Jenkins旧版[Jenkins ver. 1.653]中有以下提示；[暂时未测试]

Git Parameter Plug-In  0.8.0

Assign git tag or revision number as parameter in Parametrized builds

Warning: This plugin requires dependent plugins be upgraded and at least one of these dependent plugins claims to use a different settings format than the installed version. Jobs using that plugin may need to be reconfigured, and/or you may not be able to cleanly revert to the prior version without manually restoring old settings. Consult the plugin release notes for details.

 

## 2、新版本[指2.0以上]Dynamic Choice Parameter插件已经在官方上找不到了，官方说明存在安全漏洞；

所以使用Git Parameter Plug-In 构建参数获取分支的插件

使用方法：

Jenkins--->dev-h5-server--->配置--->参数化构建过程--->选择Git Parameter Plug-In插件：

    Name: git_branch 
    Description:描述可以写些什么 
    Parameter Type：选择Branch 
    Branch Filter:  .* 
    Tag Filter: * 
    Sort Mode:  NONE 
    Default Value:  master    #默认不选择的时候会使用master主干； 
    Selected Value: DEFAULT   #默认值为master 

其它没写上来的都留空；

源码管理--->Git--->	Branches to build

    把*/master 改成：$git_branch [就是上面定义的Name值] 

这样就可以获取到git代码分支了；

其实两个插件的方法都是差不多，只是获取出来的列表有点不一样，Dynamic Choice Parameter插件加上脚本上的切片，只保留了分支名；而Git Parameter Plug-In会把origin/都显示出来；

 

去掉那段E文的简单方法：

    cd jenkins/plugins/git-parameter/WEB-INF/lib/ 

进入到插件的目录下，创建一个临时目录：

    mkdir test 
    cp git-parameter.jar test/ 
    cd test/ 
    jar xf git-parameter.jar 
    vim ./net/uaznia/lukanus/hudson/plugins/gitparameter/GitParameterDefinition/index.properties 

把第二行删除，保存后再重新打包：

    jar cvf git-parameter.jar .* 
    /bin/cp git-parameter.jar ../     

替换原来的文件，建议修改前先备份一下；重新打包后大小从原来的64K变成了1.2M有点夸张；

重新启动Jenkins服务时就可以发现那一段已经去掉了；