---
title: Linux一些重要的配置文件之DNS文件
categories: Linux   
toc: true  
tags: [Linux重要配置文件]
---



```

/*
1.客户端DNS可以在网卡配置文件里设置
2.客户端DNS也可以在/etc/resolve.conf里配置
3.网卡里的设置DNS优先于/etc/resolve.conf
*/

#/etc/resolve.conf
[root@linux-study cys_test]# cat /etc/resolv.conf
; generated by /sbin/dhclient-script
search localdomain
nameserver 8.8.8.8
nameserver 202.106.0.20


#/ifcfg-eth0中设置
[root@lamp01 chenyansong]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=none
#HWADDR=00:0c:29:ca:6a:82
NM_CONTROLLED=yes
ONBOOT=yes
TYPE=Ethernet
#UUID="f1827545-55d8-4241-8aae-4775a48310d3"
DNS2=202.106.0.20    #设置DNS
DNS1=8.8.8.8
USERCTL=no
IPV6INIT=no
HWADDR=00:0c:29:38:6a:b1
IPADDR=192.168.0.3
NETMASK=255.255.255.0
GATEWAY=192.168.0.1

#如果我们修改了/etc/resolv.conf 中的文件，然后重启网卡：/etc/init.d/network restart, 我们发现，我们修改的东西并没有写入文件，我们使用setup去配置DNS，然后重启网卡，结果也是一样的，也是没有写入resolv.conf文件。这点我们要注意
```






