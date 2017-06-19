---
title: kvm虚拟化管理平台WebVirtMgr部署-完整记录(1)
date: 2017-06-15 14:17:27
categories:	kvm
tags: 
	- kvm
---

<!-- toc -->


 
## kvm虚拟化管理平台WebVirtMgr部署-完整记录(1)
公司机房有一台2U的服务器（64G内存，32核），由于近期新增业务比较多，测试机也要新增，服务器资源十分有限。所以打算在这台2U服务器上部署kvm虚拟化，虚出多台VM出来，以应对新的测试需求。
当KVM宿主机越来越多，需要对宿主机的状态进行调控，决定采用WebVirtMgr作为kvm虚拟化的web管理工具，图形化的WEB，让人能更方便的查看kvm 宿主机的情况和操作
WebVirtMgr是近两年来发展较快，比较活跃，非常清新的一个KVM管理平台，提供对宿主机和虚机的统一管理，它有别于kvm自带的图形管理工具（virtual machine manager），让kvm管理变得更为可视化，对中小型kvm应用场景带来了更多方便。
WebVirtMgr采用几乎纯Python开发，其前端是基于Python的Django，后端是基于Libvirt的Python接口，将日常kvm的管理操作变的更加的可视化。

## WebVirtMgr特点
操作简单，易于使用
通过libvirt的API接口对kvm进行管理
提供对虚拟机生命周期管理
WebVirtMgr 功能

## 宿主机管理支持以下功能
CPU利用率  
内存利用率  
网络资源池管理  
存储资源池管理  
虚拟机镜像  
虚拟机克隆  
快照管理  
日志管理  
虚机迁移  

## 虚拟机管理支持以下功能
CPU利用率  
内存利用率  
光盘管理  
关/开/暂停虚拟机  
安装虚拟机  
VNC console连接  
创建快照  

## 下面对部署过程进行记录，希望能帮助到有用到的朋友们。
这里我将webvirtmgr服务器和kvm服务器放在同一台机器上部署的，即单机部署  
系统：Centos 6.8  
内存:64G  
CPU:32核  
ip：192.168.1.17（内网），111.101.186.163（外网）  

## 一、首先要安装KVM虚拟化环境，参考下面的一篇博客进行安装：

kvm虚拟化管理平台WebVirtMgr部署-虚拟化环境安装-完整记录(0)  

## 二、Kvm的管理工具webvirtmgr安装和使用

### 1）安装支持的软件源
[root@openstack ops]#yum -y install http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm  

### 2）安装相关软件
---------------------------------------------------------------------------------------------------------------------  
如果报错说没有下面安装的某些软件，则下载http://pan.baidu.com/s/1bX3vkE （密码：6ucs）  
将下载的yum.tar.gz解压后的文件替换到/etc/yum.repos.d目录下，然后执行yum clean all && yum makecache  
---------------------------------------------------------------------------------------------------------------------  
[root@openstack ops]#yum -y install git python-pip libvirt-python libxml2-python python-websockify supervisor nginx  

### 3）从git-hub中下载相关的webvirtmgr代码
[root@openstack ops]#cd /usr/local/src/
[root@openstack src]#git clone git://github.com/retspen/webvirtmgr.git    （下载地址：https://pan.baidu.com/s/1pLS3kCj      获取密码：8efm）

### 4）安装webvirtmgr
[root@openstack src]#cd webvirtmgr/  
[root@openstack webvirtmgr]#pip install -r requirements.txt  

### 5）安装数据库
[root@openstack webvirtmgr]#yum install python-sqlite2   

### 6）对django进行环境配置
[root@openstack webvirtmgr]#pwd  
/usr/local/src/webvirtmgr   
[root@openstack webvirtmgr]#./manage.py syncdb           //默认是python执行，如下报错，换用其他版本的python  
-------------------------------------------------- ----------------------------   
注意此处用默认的python执行上面命令，一般会报错，如下：  
ImportError: No module named django.core.management   

### 这个一般是由于python版本引起的，因为系统自带有好几个版本的python 
[root@openstack webvirtmgr]# python       //按Tab键自查找  
python python2.6  
python2 python2.6-config python-config   
[root@openstack webvirtmgr]# python -V  
Python 2.6.6  

由此可看出，系统默认的Python版本是2.6.6  
说明上面命令默认是python2.6执行的  

既然使用python2.6执行上面的命令报错，那就换用其他版本python2执行（如果当前是python3.3.0，那么就将下面的/usr/bin/python2换成/usr/bin/python2.6）  
[root@openstack webvirtmgr]#/usr/bin/python2 manage.py syncdb     //最终发现使用python2执行这个命令就不报错了  
............  
............  
You just installed Django's auth system, which means you don't have any superusers defined.  
Would you like to create one now? (yes/no): yes  
Username (leave blank to use 'root'): admin  
Email address: wangshibo@163.com  
Password:*********  
Password (again):*********  
--------------------- --------------------- --------------------- --------------------- 
[root@openstack webvirtmgr]#/usr/bin/python2 manage.py collectstatic          //生成配置文件（同样使用python2版本执行，不要使用默认的python执行）
WARNING:root:No local_settings file found.

You have requested to collect static files at the destination
location as specified in your settings.

This will overwrite existing files!
Are you sure you want to do this?

Type 'yes' to continue, or 'no' to cancel: yes
..........
..........

[root@openstack webvirtmgr]#/usr/bin/python2 manage.py createsuperuser  //添加管理员账号（同样使用python2版本执行，不要使用默认的python执行）
WARNING:root:No local_settings file found.
Username: ops                                                   //这个是管理员账号，用上面的admin和这个管理员账号都可以登陆webvirtmgr的web界面管理平台
Email address: wangshibo@163.com
Password:
Password (again):
Superuser created successfully.

## 7）拷贝web到 相关目录
[root@openstack ops]#mkdir -pv /var/www
[root@openstack ops]#cp -Rv /usr/local/src/webvirtmgr /var/www/webvirtmgr

## 8）设置ssh
[root@openstack ops]#ssh-keygen -r rsa             //产生公私钥
[root@openstack ops]#ssh-copy-id 192.168.1.17        //由于这里webvirtmgr和kvm服务部署在同一台机器，所以这里本地信任。如果kvm部署在其他机器，那么这个是它的ip
[root@openstack ops]#ssh 192.168.1.17 -L localhost:8000:localhost:8000 -L localhost:6080:localhost:60

## 9）编辑nginx配置文件
提前确保/etc/nginx/nginx.conf文件里开启了“include /etc/nginx/conf.d/*.conf;”

[root@openstack ops]#vim /etc/nginx/conf.d/webvirtmgr.conf           //添加下面内容到文件中  
server {
listen 80 default_server;

server_name $hostname;
#access_log /var/log/nginx/webvirtmgr_access_log;

location /static/ {
root /var/www/webvirtmgr/webvirtmgr; # or /srv instead of /var
expires max;
}

location / {
proxy_pass http://127.0.0.1:8000;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
proxy_set_header Host $host:$server_port;
proxy_set_header X-Forwarded-Proto $remote_addr;
proxy_connect_timeout 600;
proxy_read_timeout 600;
proxy_send_timeout 600;
client_max_body_size 1024M; # Set higher depending on your needs
}
}

[root@openstack ops]#mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak

## 10）启动nginx
[root@openstack ops]#/etc/init.d/nginx restart

## 11）修改防火墙规则
[root@ops ~]#vim /etc/sysconfig/selinux
......
SELINUX=disabled

[root@ops ~]#setenforce 0
setenforce: SELinux is disabled
[root@ops ~]#getenforce
Disabled

[root@openstack ops]#/usr/sbin/setsebool httpd_can_network_connect true

## 12）设置 supervisor (如果iptables防火墙开启的话，就必须要开通80、8000、6080端口访问)
[root@openstack ops]#chown -R nginx:nginx /var/www/webvirtmgr

[root@openstack ops]#vim /etc/supervisord.conf     //在文件末尾添加,注意将默认的python改为python2，因为上面只有用这个版本执行才不报错！
[program:webvirtmgr]
command=/usr/bin/python2 /var/www/webvirtmgr/manage.py run_gunicorn -c /var/www/webvirtmgr/conf/gunicorn.conf.py                     //启动8000端口
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
logfile=/var/log/supervisor/webvirtmgr.log
log_stderr=true
user=nginx

[program:webvirtmgr-console]
command=/usr/bin/python2 /var/www/webvirtmgr/console/webvirtmgr-console                               //启动6080端口（这是控制台vnc端口）
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=nginx
[root@openstack ops]#vim /var/www/webvirtmgr/conf/gunicorn.conf.py    //确保下面bind绑定的是本机的8000端口，这个在nginx配置中定义了，被代理的端口
bind = '127.0.0.1:8000'
## 13）设置开机启动
[root@openstack ops]#chkconfig supervisord on
[root@openstack ops]#vim /etc/rc.local
/usr/sbin/setsebool httpd_can_network_connect true

## 14）启动进程
[root@openstack ops]#/etc/init.d/supervisord restart
15）查看进程
[root@openstack ops]#netstat -lnpt    //即可以看到6080和8000已经启动
[root@openstack ops]# lsof -i:6080
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
python2 53476 nginx 3u IPv4 364124 0t0 TCP *:6080 (LISTEN)
[root@openstack ops]#lsof -i:8000

------------------------------------------------------------------------------
一般来说，只要上面的supervisord服务启动了，8000和6080端口就起来了。
因为在/etc/supervisord.conf文件里配置了这两个端口的启动设置

如果supervisord服务启动了，8000和6080端口还是没起来！
那么就需要手动启动/etc/supervisord.conf文件里配置的这两个端口的启动命令，即：
[root@openstack ops]# /usr/bin/python2 /var/www/webvirtmgr/manage.py run_gunicorn -c /var/www/webvirtmgr/conf/gunicorn.conf.py       //这个命令会一直在操作
[root@openstack ops]# /usr/bin/python2 /var/www/webvirtmgr/console/webvirtmgr-console                        //这个命令会一直在操作
[root@openstack ops]# /etc/init.d/supervisord restart
[root@openstack ops]# lsof -i:8000
[root@openstack ops]# lsof -i:6080
------------------------------------------------------------------------------

## 16）web访问
http://111.101.186.163/login/

这里用超级管理员登陆，只有超级管理员登陆后才能看到“基础构架”窗口

普通用户登陆后，只能看到“WebVirtMgr”一个窗口

 

选择“SSH链接“，设置Label，IP，用户

注意：Label与IP要相同

打开后，有报错！看来在上面使用ssh连接的配置环节有误所致！

解决措施：

## 1)在webvirtmgr服务器（服务端）上（这里kvm和WebVirtMgr部署在同一台机器上）创建nginx用户家目录（默认nginx服务安装时是没有nginx家目录的），生成nginx的公私钥
[root@openstack ops]# cd /home/
[root@openstack home]# mkdir nginx
[root@openstack home]# chown nginx.nginx nginx/
[root@openstack home]# chmod 700 nginx/ -R
[root@openstack home]# su - nginx -s /bin/bash
-bash-4.1$ ssh-keygen                             #期间输入yes后直接回车，回车
-bash-4.1$ touch ~/.ssh/config && echo -e "StrictHostKeyChecking=no\nUserKnownHostsFile=/dev/null" >> ~/.ssh/config
-bash-4.1$ chmod 0600 ~/.ssh/config

## 2)在kvm（客服端）服务器上（这里kvm和WebVirtMgr部署在同一台机器上）配置用户，这里默认采用root用户
---------------------------------------------------------------------------------------------------------------------
如果采用其他用户，比如webvirtmgr，操作如下:
[root@openstack ops]#useradd webvirtmgr
[root@openstack ops]#echo "123456" | passwd --stdin webvirtmgr
[root@openstack ops]#groupadd libvirt
[root@openstack ops]#usermod -G libvirt -a webvirtmgr
---------------------------------------------------------------------------------------------------------------------

## 3)在webvirtmgr服务器（服务端）上（这里kvm和WebVirtMgr部署在同一台机器上），将nginx用户的ssh-key上传到kvm服务器上（这里kvm和WebVirtMgr部署在同一台机器上）
[root@openstack ops]# su - nginx -s /bin/bash
-bash-4.1$ ssh-copy-id root@192.168.1.17
Warning: Permanently added '192.168.1.17' (RSA) to the list of known hosts.
root@192.168.1.17's password: #输入192.168.1.17即本机的root账号
Now try logging into the machine, with "ssh 'root@192.168.1.17'", and check in:

.ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
---------------------------------------------------------------------------------------------------------------------
这里采用的是root用户，如果采用其他用户，比如上面假设的webvirtmgr用户，操作如下:
[root@openstack ops]#su - nginx -s /bin/bash
-bash-4.1$ssh-copy-id webvirtmgr@192.168.0.23
---------------------------------------------------------------------------------------------------------------------

## 4）在kvm（客服端）服务器上（这里kvm和WebVirtMgr部署在同一台机器上）配置 libvirt ssh授权
[root@openstack ops]# vim /etc/polkit-1/localauthority/50-local.d/50-libvirt-remote-access.pkla
[Remote libvirt SSH access]
Identity=unix-user:root #注意这里采用的是root用户
Action=org.libvirt.unix.manage
ResultAny=yes
ResultInactive=yes
ResultActive=yes

[root@openstack ops]# chown -R root.root /etc/polkit-1/localauthority/50-local.d/50-libvirt-remote-access.pkla
-------------------------------------------------------------------------------------------------------------------------------
这里采用的是root用户，如果采用其他用户，比如上面假设的webvirtmgr用户，操作如下:
[root@openstack ops]#vim /etc/polkit-1/localauthority/50-local.d/50-libvirt-remote-access.pkla
[Remote libvirt SSH access]
Identity=unix-user:webvirtmgr #这里就设定webvirtmgr用户
Action=org.libvirt.unix.manage
ResultAny=yes
ResultInactive=yes
ResultActive=yes

[root@openstack ops]#chown -R webvirtmgr.webvirtmgr /etc/polkit-1/localauthority/50-local.d/50-libvirt-remote-access.pkla
--------------------------------------------------------------------------------------------------------------------------------

## 5）重启 libvirtd 服务
/etc/init.d/libvirtd restart

这样上面报错的问题就迎仍而解了！

然后重新ssh方式连接就ok了，就不会有上面那个报错了~

但是，又出现了其他报错（如下）！尼玛～～  接续排查！

解决措施：

 在WebVirtMgr服务器本地使用ssh方式连接，在终端命令行里：

[root@openstack .ssh]# virsh -c qemu+ssh://103.10.86.17/system list
The authenticity of host '103.10.86.17 (103.10.86.17)' can't be established.
RSA key fingerprint is 3d:c1:2e:70:e9:e5:1d:84:40:a2:63:82:af:e5:cc:cd.
Are you sure you want to continue connecting (yes/no)? yes
error: End of file while reading data: Warning: Permanently added '103.10.86.17' (RSA) to the list of known hosts.: Input/output error
error: failed to connect to the hypervisor

看日志 tail /var/log/secure | grep sshd 发现是我这里主动发出断开的.有可能检测到libvirtd有些问题导致的。
当时使用virt-manage可以查询到远程的信息.估计是sshd出现的问题把.

折腾一会，暂时没找到解决方案

决定先选用通过tcp协议进行迁移的（但是这种方式没有用ssh连接方式安全——）
等后面有时间了，再想办法解决上面ssh方式连接的错误吧


使用tcp进行对远程libvirtd进行连接访问的配置如下：

1）修改文件/etc/sysconfig/libvirtd，用来启用tcp的端口
[root@openstack ops]# cat /etc/sysconfig/libvirtd
........
LLIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf
LIBVIRTD_ARGS="--listen"

2)修改文件/etc/libvirt/libvirtd.conf
[root@openstack ops]#vim /etc/libvirt/libvirtd.conf
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
auth_tcp = "none"

3)运行 libvirtd
[root@openstack ops]#service libvirtd restart

如果没起效果(我的就没有生效，那么使用命令行.如果生效了，执行下面命令就会报错
[root@openstack ops]#libvirtd --daemon --listen --config /etc/libvirt/libvirtd.conf

4)查看运行进程
[root@openstack ops]# ps aux | grep libvirtd
root 16563 1.5 0.1 925880 7056 ? Sl 16:01 0:28 libvirtd -d -l --config /etc/libvirt/libvirtd.conf

5)查看端口
[root@openstack ops]# lsof -i:16509

6）在source host连接dest host远程libvirtd查看信息（也可以用公网ip）
[root@openstack ops]# virsh -c qemu+tcp://192.168.1.17/system
Welcome to virsh, the virtualization interactive terminal.

Type: 'help' for help with commands
'quit' to quit

virsh #

成功使用tcp去访问libvirtd。

注意：

在使用tcp方式连接后，会出现连接终端的情况！

[root@openstack .ssh]# virsh -c qemu+tcp://192.168.1.17/system
error: Cannot recv data: Connection reset by peer
error: failed to connect to the hypervisor

连接断开，重新连接便可。

[root@openstack ops]# ps aux | grep libvirtd
root 59619 0.6 0.0 1008128 22048 ? Sl 19:17 0:06 libvirtd --daemon --config /etc/libvirt/libvirtd.conf --listen
root 61081 0.0 0.0 103316 1004 pts/2 S+ 19:33 0:00 grep --color libvirtd
[root@openstack ops]# kill -9 59619
[root@openstack ops]# ps aux | grep libvirtd
root 61083 0.0 0.0 103312 904 pts/2 S+ 19:33 0:00 grep --color libvirtd
[root@openstack ops]# libvirtd --daemon --listen --config /etc/libvirt/libvirtd.conf
[root@openstack ops]# ps aux | grep libvirtd
root 61086 13.5 0.0 418240 6576 ? Sl 19:33 0:00 libvirtd --daemon --listen --config /etc/libvirt/libvirtd.conf
root 61176 0.0 0.0 103312 908 pts/2 S+ 19:33 0:00 grep --color libvirtd
[root@openstack ops]# virsh -c qemu+tcp://192.168.1.17/system
Welcome to virsh, the virtualization interactive terminal.

Type: 'help' for help with commands
'quit' to quit

virsh #

后续发现，webvirtmgr连上后，过一会儿就会断开！

针对这个情况，可以可以写个定时脚本，如下：

[root@openstack ops]# cat /usr/local/src/libvirtd.sh

#!/bin/bash
ps -ef | grep "libvirtd --daemon --listen"|grep -v grep|awk -F" " '{print $2}'|xargs kill -9
/usr/sbin/libvirtd --daemon --listen --config /etc/libvirt/libvirtd.conf

[root@openstack ops]# crontab -l
* * * * * /bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1
* * * * * sleep 10;/bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1
* * * * * sleep 20;/bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1
* * * * * sleep 30;/bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1
* * * * * sleep 40;/bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1
* * * * * sleep 50;/bin/bash -x /usr/local/src/libvirtd.sh > /dev/null 2>&1

*********************************************************************************

一般如上配置后，webvirtmgr里的控制台是可以正常连接虚拟机的。

但如果webvirtmgr里通过控制台页面(vnc)连接虚拟机失败，可以按照下面的操作方法尝试解决：

kvm原来的安装方式是客户端需要安装vncviewer，才能看到安装页面，而webvirtmgr使用了novnc，页面通过websocket进行通信。

1）首先需要安装novnc
[root@openstack ops]# yum install -y novnc

2）防火墙打开vnc的6080端口
[root@openstack ops]# vim /etc/sysconfig/iptables
.......
-A INPUT -p tcp -m state --state NEW -m tcp --dport 6080 -j ACCEPT
.......

[root@openstack ops]# /etc/init.d/iptables restart

3）由上面可知，webvirtmgr进程通过supervisor管理。这里需要重启supervisor进程
[root@openstack ops]# /etc/init.d/supervisord restart

这样，再次打开Webvirtmgr的控制台，发现就能连上虚拟机了！

******************
可以在webvirtmgr服务器上通过命令行尝试下连接：
[root@openstack ops]# novnc_server --help
Usage: novnc_server [--listen PORT] [--vnc VNC_HOST:PORT] [--cert CERT]

Starts the WebSockets proxy and a mini-webserver and
provides a cut-and-paste URL to go to.

--listen PORT Port for proxy/webserver to listen on
Default: 6080
--vnc VNC_HOST:PORT VNC server host:port proxy target
Default: localhost:5900
--cert CERT Path to combined cert/key file
Default: self.pem
--web WEB Path to web files (e.g. vnc.html)
Default: ./

[root@openstack ops]# novnc_server --listen 192.168.1.17:6086           #端口在6080后的都可以用
Warning: could not find self.pem
Starting webserver and WebSockets proxy on port 192.168.1.17:6086
WARNING: no 'numpy' module, HyBi protocol will be slower
WebSocket server settings:
- Listen on 192.168.1.17:6086
- Flash security policy server
- Web server. Web root: /usr/share/novnc
- No SSL/TLS support (no cert file)
- proxying from 192.168.1.17:6086 to localhost:5900


Navigate to this URL:

http://kvm-server:192.168.1.17:6086/vnc.html?host=kvm-server&port=192.168.1.17:6086

Press Ctrl-C to exit

*********************************************************************************
***************当你发现自己的才华撑不起野心时，就请安静下来学习吧***************