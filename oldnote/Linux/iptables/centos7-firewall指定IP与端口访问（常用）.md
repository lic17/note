[TOC]



基本使用

启动：`systemctl start firewalld`

关闭：`systemctl stop firewalld`

查看状态：`systemctl status firewalld`

开机禁用：`systemctl disable firewalld`

开机启用：`systemctl enable firewalld`

配置firewalld-cmd

查看版本：`firewall-cmd --version`

查看帮助：`firewall-cmd --help`

显示状态：`firewall-cmd --state`

查看所有打开的端口：`firewall-cmd --zone=public --list-ports`

更新防火墙规则：`firewall-cmd --reload`

端口开放

添加：`firewall-cmd --zone=public --add-port=80/tcp --permanent`

重新载入：`firewall-cmd --reload`

删除：`firewall-cmd --zone= public --remove-port=80/tcp --permanent`

端口转发

添加（例如3306 -> 3336）：

```
firewall-cmd --permanent --zone=public --add-forward-port=port=3336:proto=tcp:toport=3306:toaddr=
```

删除：

```
firewall-cmd --permanent --remove-forward-port=port=3306:proto=tcp:toport=3336:toaddr=
```

查看转发的端口：`firewall-cmd --list-forward-ports`

查看当前开了哪些端口

其实一个服务对应一个端口，每个服务对应/usr/lib/firewalld/services下面一个xml文件。

firewall-cmd --list-services

查看还有哪些服务可以打开

firewall-cmd --get-services

查看所有打开的端口： 

firewall-cmd --zone=public --list-ports

更新防火墙规则： 

firewall-cmd --reload

 

```shell
# 添加多个端口
firewall-cmd --permanent --zone=public --add-port=8080-8083/tcp 

# 删除某个端口
firewall-cmd --permanent --zone=public --remove-port=81/tcp 

# 针对某个 IP开放端口
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.142.166' port protocol='tcp' port='6379' accept"
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.0.233' accept" 

# 删除某个IP
firewall-cmd --permanent --remove-rich-rule='rule family='ipv4' source address='192.168.1.51' accept" 

# 针对一个ip段访问
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.10.0/24' accept"
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.1.0/24' port protocol='tcp' port='9200' accept"

#mac地址
firewall-cmd --zone=work --add-source=00:11:22:33:44:55
firewall-cmd --zone=work --add-rich-rule='rule source mac=11:22:33:44:55:66 drop'

#iptables的设置方式
iptables -I INPUT -m mac --mac-source 00:16:31:D7:3F:63  -j ACCEPT



# 添加操作后别忘了执行重载
firewall-cmd --reload
```

　　

添加本地的

 

## firewalld报错问题

```shell
#如下的报错
Debian Bug report logs - #914694
firewall-cmd --reload fails: RULE_REPLACE failed (No such file or directory): rule in chain {INPUT,OUTPUT}


#解决的方式：设置为yes
root@soc-PC:/home/workspaces/apache-flume-1.8.0-bin# cat /etc/firewalld/firewalld.conf|grep IndividualCalls
# IndividualCalls
IndividualCalls=yes
```





# iptables-save

```shell
iptables-save > /etc/sysconfig/iptables
iptables-restore </etc/sysconfig/iptables 
```





#  防火墙脚本实例

```shell
#!/bin/bash
systemctl stop firewalld
\cp -p /usr/lib/firewalld/zones/drop.xml /etc/firewalld/zones/
systemctl start firewalld
firewall-cmd --set-default-zone=drop
firewall-cmd --permanent --zone=drop --change-interface=ens32
firewall-cmd --permanent --zone=drop --add-service=https
firewall-cmd --permanent --zone=drop --add-protocol=icmp
firewall-cmd --permanent --zone=drop --add-masquerade
firewall-cmd --permanent --zone=drop --add-rich-rule="rule family="ipv4" source address="192.168.2.208" port protocol="tcp" port="5210" accept"
firewall-cmd --permanent --zone=drop --add-rich-rule="rule family="ipv4" source address="192.168.2.206" port protocol="tcp" port="5210" accept"
firewall-cmd --permanent --zone=drop --add-rich-rule="rule family="ipv4" source address="116.226.230.115" port protocol="tcp" port="8023" accept"
firewall-cmd --reload
```







