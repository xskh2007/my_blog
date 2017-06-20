---
title: git crlf换行符的问题解决
date: 2017-06-15 14:17:27
categories:	git
tags: 
	- git
---

<!-- toc -->

# git crlf换行符的问题解决

## git使用中遇到的换行符问题总结

#### 问题描述
项目组现在用git做版本控制，使用中遇到不同平台下换行符不同造成的问题，windows下的换行符为crlf，linux和MAX OS 下换行符是 lf。linux和MAX os就按说明设置为```core.autocrlf input```（貌似是默认值），windows设置为```core.autocrlf true```。可是有时候还是会遇到换行符的问题。review的时候就会发现有的commit的变化是所有行都被删除重建。

#### git中设置core.autocrlf####

假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。

Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。用`core.autocrlf`来打开此项功能，如果是在Windows系统上，把它设置成`true`，这样当签出代码时，LF会被转换成CRLF：

$ git config --global core.autocrlf true

Linux或Mac系统使用LF作为行结束符，因此你不想 Git 在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把`core.autocrlf`设置成input来告诉 Git 在提交时把CRLF转换成LF，签出时不转换：

$ git config --global core.autocrlf input

这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。

如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置`false`取消此功能，把回车符记录在库中：

$ git config --global core.autocrlf false

#### 解决方法

1. 修改git设置 core.autocrlf=input.检出时不转换，提交转换为lf，这样可以避免提交windows换行符的情况；
2. 修改eclipse设置 windows>General> workspace  下 new text file line delimiter 选择Unix。
3. 已有的项目可能已经存在换行符不同的问题需要修正一下。
如果当前开发有多个分支且各分支不同步，需要每个分支进行一次转换：选中项目  file> convert line delimiter to > Unix ，创建新的commit；
如果只有一个分支或多个分支处于同一节点。可以从master切换一个新分支，进行第2步的修改操作，然后commit ，将此分支合并到所有分支。
4.  将修改过的分支push到gitlab，其他成员更新代码即可。
(ps:由于每个人系统不同或者就是git的问题，可能出现更新完代码换行符不变，这时以服务器上的代码为准重新clone一份最新代码即可)