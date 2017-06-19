---
title: 用iptables实现单网卡做网关上网
date: 2017-06-15 14:17:27
categories:	iptables
tags: 
	- iptables
---

<!-- toc -->







测试环境，配置iptables的服务器A的ip：172.16.81.156，默认网关172.16.80.1；网段192.168.1.0/24通过交换机与A机连通；
A主机只有一块网卡eth0，主要的配置就是在单网卡上绑定一个虚拟ip，在通过iptables规则将192.168.1.0/24段的源ip进行伪装，达到上网目的。
配置如下：
/etc/sysconfig/ifcfg-eth0的配置信息：
DEVICE=eth0
BOOTPROTO=static
IPADDR=172.16.81.156
NETMASK=255.255.248.0
GATEWAY=172.16.80.1
NETWORK=172.16.80.0
ONBOOT=yes
TYPE=Ethernet

/etc/sysconfig/ifcfg-eth0:1的配置信息：
DEVICE=eth0:1
BOOTPROTO=static
IPADDR=192.168.1.1
NETMASK=255.255.255.0
ONBOOT=yes

echo 1 > /proc/sys/net/ipv4/ip_forward(这句写到/etc/rc.d/rc.local中)

iptables中的规则：
然后加入以下规则
iptables -F
iptables -F -t nat
iptables -A INPUT -i eth0 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -j MASQUERADE
iptables -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
service iptables save（自动生成/etc/sysconfig/iptables）

客户端配置：
简单的说，就是在192.168.1.0/24这个网段中的一个机器，指定以192.168.1.1为网关，再添加可用的DNC就可以了.
linux客户端的配置方法如下：
ifcfg-eth0：
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.2
NETMASK=255.255.255.0
GATEWAY＝192.168.1.1
ONBOOT=yes
设置DNS(/etc/resolv.con)
重启机器就可以上网了。