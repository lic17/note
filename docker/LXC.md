[TOC]



主机虚拟化

​	资源开销大

减少中间层

虚拟机是为了环境的隔离

用户空间的隔离

​	jail

vserver(chroot)

主机名，域名：UTS

根文件系统：Mount

每一个用户空间：IPC

进程应该从属于父进程(独立的进程树)pid：init或者从属于init

运行的进程，应该以某个用户的身份运行，必须为每个用户空间伪装为root，在宿主机的系统上只是一个普通的用户

每一个用户空间，都有一个网卡，网络接口，有自己TCP/IP协议栈（网络）

名称空间实现上述功能隔离

内核资源的名称

所以需要和linux内核版本进行对应

* Linux Namespaces

  ![1562221249555](E:\git-workspace\note\images\docker\lxc1.png)





限制每个用户空间的资源

整体资源上，做比率性分配

单一资源上，做核心绑定分配（只能用几个CPU，多少内存）

cgroups(Control groups)

内存是不可压缩资源：要就必须有

CPU是可压缩，可以实现

Control Groups

将系统资源分配到多个组，然后将这些资源分配到用户空间

![1562222341834](E:\git-workspace\note\images\docker\lxc2.png)

控制组：用户空间的资源分配

容器的隔离能力（安全性）：Selinux加固用户空间的边界



LXC(LinuX Container)

template:自动实现安装过程

Docker是LXC的增强版

* 大规模创建容器
* LXC的二次封装，发行版
* 利用LXC做容器，镜像技术



在一个容器中只运行一个进程，nginx只运行在nginx的容器中，这样可以最小化定义，共享的文件需要多份，调试工具也应该准备多份（每个容器），自带调试工具

容器给运维带来了极大的不便，但是给开发带来了极大的遍历（一次编写，到处运行），随便部署

容器编排工具(发布)，手动管理麻烦

运维的核心是：维稳

为每一个镜像自带调试工具

批量下载容器

分层构建，联合挂载

容器不需要持久，从创建开始，停止结束

容器的启动，调度-->容器编排工具

docker编排工具：machine + swarm + compose

mesos统一资源调度+marachon

kubernetes -> k8s 容器编排工具

Moby - CE

google-> CNCF 

libcontainer -> runC  (容器运行时的环境标准)

docker架构

![1562240223857](E:\git-workspace\note\images\docker\arch1.png)



Registry

​	repository： 每个仓库放一种应用程序：仓库名是应用程序名，如：nginx，而nginx有很多的版本

每一个镜像有一个tag:如：nginx:1.10, nginx:1.15， nginx:latest, nginx:stable

仓库名+tag唯一的标识镜像



镜像是静态的，不会运行，容器是动态，他有生命周期

images,container, networks volumes, plugins 对象，实现增删改查

![1562242314389](E:\git-workspace\note\images\docker\instruc2.png)







