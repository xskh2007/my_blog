---
title: PowerCLI 批量克隆，批量配ip，批量重启
date: 2017-06-15 14:17:27
categories:	PowerCLI
tags: 
	- PowerCLI
---

<!-- toc -->



### PowerCLI 批量克隆，批量配ip，批量重启


	#          Welcome to VMware vSphere PowerCLI!
	#
	#Log in to a vCenter Server or ESX host:              Connect-VIServer
	#To find out what commands are available, type:       Get-VICommand
	#To show searchable help for all PowerCLI commands:   Get-PowerCLIHelp
	#Once you've connected, display all virtual machines: Get-VM
	#If you need more help, visit the PowerCLI community: Get-PowerCLICommunity
	
	
	$vc = '172.17.1.2'
	Connect-VIServer -Server $vc -username "administrator" -Password "12345"
	
	$vmhost="172.17.1.10"
	$template="172.17.2.12.tomcat_template"
	$datastore="datastore2 (2)"
	#$datastore="datastore2 (1)"
	#$custsysprep = Get-OSCustomizationSpec myCustSpec
	$resourcepool="server1"
	$sourcefile="C:\powercli\vmlist1.txt"
	$desfile="/tmp/"
	$vm_list="C:\powercli\vmlist2.txt"
	#Copy-VMGuestFile -Source C:\powercli\vmlist1.txt -Destination /tmp/ -VM 172.17.2.101_weapon -LocalToGuest -GuestUser nbj -GuestPassword Zzjr@2016
	
	function MyCopyFile()
	{
	for($file=[IO.File]::OpenText($vm_list);!($file.EndOfStream);$line=$file.ReadLine() )
	{
		#$line;
		#New-vm -vmhost $vmhost -Name $line -Template $template -Datastore $datastore -ResourcePool $resourcepool
		#start-vm $line
		CreateEth0($line)
		Copy-VMGuestFile -Source C:\powercli\ifcfg-eth0 -Destination /etc/sysconfig/network-scripts/ -VM $line -LocalToGuest -Force -GuestUser root -GuestPassword Zzjr#2016
		Copy-VMGuestFile -Source C:\powercli\70-persistent-net.rules -Destination /etc/udev/rules.d/ -VM $line -LocalToGuest -Force -GuestUser root -GuestPassword Zzjr#2016
		#stop-vm $line
		#Copy-VMGuestFile -Source $Sourcefile -Destination 
		#Copy-VMGuestFile -Source C:\powercli\vmlist1.txt -Destination /tmp/ -VM 172.17.2.101_weapon  -GuestUser nbj -GuestPassword Zzjr@2016
	
	}
	$file.Close()
	}
	
	function CreateEth0($vmname)
	{
	$str = @"
	DEVICE=eth0
	TYPE=Ethernet
	ONBOOT=yes
	NM_CONTROLLED=yes
	BOOTPROTO=static
	NETMASK=255.255.255.0
	GATEWAY=172.17.2.1
	"@
	
	$ipaddr="IPADDR="
	$ip=$vmname -replace "_.*$"
	$ip_str=$ipaddr + $ip
	$str|Out-File -filepath C:\powercli\ifcfg-eth0 -Encoding ascii 
	$ip_str|Out-File  -filepath C:\powercli\ifcfg-eth0 -Encoding ascii -Append
	[IO.File]::ReadAllText("C:\powercli\ifcfg-eth0") -replace "`r`n", "`n"|Out-File -filepath C:\powercli\ifcfg-eth0 -Encoding ascii
	$rules="#create by qiantu"
	$rules|Out-File  -filepath C:\powercli\70-persistent-net.rules -Encoding ascii
	}
	
	
	function restart()
	{
	for($file=[IO.File]::OpenText($vm_list);!($file.EndOfStream);$line=$file.ReadLine() )
	{
		#$line;
		#New-vm -vmhost $vmhost -Name $line -Template $template -Datastore $datastore -ResourcePool $resourcepool
		restart-vm $line
		#stop-vm $line
		#Copy-VMGuestFile -Source $Sourcefile -Destination 
		#Copy-VMGuestFile -Source C:\powercli\vmlist1.txt -Destination /tmp/ -VM 172.17.2.101_weapon  -GuestUser nbj -GuestPassword Zzjr@2016
	
	}
	$file.Close()
	}
	
	restart
	#MyCopyFile
	