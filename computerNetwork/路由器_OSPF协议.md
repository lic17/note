[TOC]

# OSPF协议概述

RIP协议的不足：

1. 周期性的发送整张路由表，在大型的自治网络中会消耗可观的网络带宽
2. 由于RIP最多只能支持网络之间15个跳步，限制了网络的规模
3. RIP计算最佳路径只考虑最小跳数，而不考虑网络的带宽，可靠性，延迟等因素，造成RIP选择的路径不见得是最优的路径
4. RIP的收敛非常慢，并且可能造成路由自环问题



OSPF的代价值的计算公式：$10^8/(链路带宽)$，对于100Mbps快速以太网，其代价值为1，对于10Mpbs的以太网，其代价值为10，对于serial接口1.544Mbps，则代价值为64，**代价值越小，说明路径越优**



# OSPF的工作原理

运行OSPF协议的路由器首先手机其所在网络区域上各路由器的连接状态信息，即链路状态信息，从而生成链路状态数据库LSDB，路由器通过链路状态数据库LSDB掌握了该区域上所有路由器的链路状态信息，也就等于了解了整个网络的拓扑状况，然后OSPF路由器利用最短路径有限SPF算法，独立的计算出到达任意网络的路由，当网络拓扑结构发生变化的时候，运行OSPF协议的路由器迅速的发出链路状态信息，通知到网络中同区域的所有路由器，从而使得所有的路由器更新自己的链路状态数据库，每台路由器根据SPF算法重新计算到达任意网络的最佳路由，从而更新自己的路由表

![image-20190907110203580](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907110203580.png)

# OSPF的特点

* 适应范围：支持各种规模的网络，最多支持上千路由器，同时OSPF也支持可变长子网掩码VLSM
* 快速收敛：在网络的拓扑结构发生变化后能够立即发送链路状态信息的更新报文(他不像RIP，周期性的发送报文，他只会在拓扑结构发生变化的时候更新，协议带来的网络开销很小)，这样使得网络拓扑改变后，能够迅速收敛
* 无自环：由于OSPF根据收集到的链路状态用最短路径树算法计算路由，从算法本身保证了不会生成自环路由
* 区域划分：允许自治系统的网络被划分为区域进行管理，从而减少了占用的网络带宽
* 支持验证：支持基于接口的报文验证，以保证路由计算的安全性，也可以防止对路由器，路由协议的攻击行为，同时OSPF数据包直接封装于IP协议之上(RIP协议是应用层协议，外层还有UDP的封装，但是OSPF是传输层协议)



# OSPF的基础概念

* 路由器ID

  1. 路由器ID长度为32位，用于标识OSPF区域内的每一台路由器，这一个编号在整个自治系统内是**唯一的**
  2. 路由器ID要求稳定，不能随便更改，因为每台路由器都维护了其他路由器的链路状态数据，如果一台路由器的ID改变了，那么其他所有的路由器都会更新这个路由器的链路状态数据，通常会采用路由器上处于激活(UP)状态的物理接口中IP地址最大的那个接口的IP地址作为路由器ID，如果配置了逻辑回环接口(Lookback interface),则采用具有最大IP地址的环回接口的IP地址作为路由器ID，采用环回接口的好处是，他不像物理接口那样随时可能失效，环回接口不会像物理接口那样down掉，因此，用环回接口的IP地址作为路由器的ID更稳定，也更可靠

* OSPF区域

  * 如果不划区域的话，存在下面的问题

  	1. 大规模的网络中，链路状态数据库LSDB会非常大，消耗大量的网络带宽，同时增加了网络中路由器的存储压力
  	2. 由于SPF算法的复杂性，会造成路由器的CPU负担增大
  	3. 由于庞大的网络出现故障的可能性增加，如果一台路由器的状态发生变化就可能造成整个网络所有的路由器的SPF重新计算
	
  	为了解决上述问题：OSPF提出了区域的概念，也就是将运行OSPF协议的路由器分成若干个区域，缩小可能出现问题的范围，链路状态信息指挥在每个区域内部泛洪，这样就减少了链路状态数据库LSDB的大小，也减轻了整个路由器失效对网络整体的影响，当网络中拓扑发生变化的时候，可以大大加速路由器收敛过程
	
  * 区域是在自治系统内部由网络管理员人为划分，并使用区域ID进行标识，OSPF区域ID长度32位，可以使用十进制数的格式来定义，如区域0，也可以使用IP地址的格式，如区域0.0.0.0，OSPF还规定，如果划分了多个区域，那么必须有一个区域0，称为骨干区域，其他类型的区域要与骨干区域相连
  
  * 区域内路由器IAR
  
  * 区域边界路由器ABR
  
  * 自治系统边界路由器ASBR
  
    ![image-20190907114251096](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907114251096.png)
  
  
  
  边界路由器，具有他连接的所有的边界链路状态数据库

* 链路状态公告

  链路状态公告LSA是链路状态信息的统称，也就是说链路状态数据库LDSB实际上是由链路状态公告LSA条目组成，链路状态公告LSA条目是链路状态数据库LSDB的基本元素
  
  ![image-20190907192800525](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907192800525.png)

* 指定路由器DR，备份指定路由器BDR，非指定路由器DRother

  **为了减少泛洪的数据量，我们采用DR和BDR**

  ![image-20190907192958056](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907192958056.png)

  * DR的选择原则：路由器优先级高的路由器将被选举为DR，网络中的所有路由器的优先级默认为1，最大为255，在路由器优先级相同的情况下，具有最大路由器ID的路由器将成为DR

  * BDR的选择原则：路由器优先级次高的被选定，如果路由器优先级相同的情况下，具有第二大路由器ID的路由器将成为BDR，如果路由器优先级为0，则表示路由器不参加DR、BDR选举过程，也不会成为DR，BDR

  * 除了DR，BDR之外的所有的路由器成为非指定路由器DRother

* OSPF中邻居关系和邻接关系

  DRother之间形成邻居关系（Neighbors)，DRother与DR、BDR之间不但是邻居关系还是邻接关系（Adjacency)

  ![image-20190907193938374](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907193938374.png)

  邻居关系的路由器之间只会定时传递OSPF的问候hello报文

  邻接关系的路由器之间不但定时传递OSPF的问候hello报文，**同时还可以发送链路状态公告LSA泛洪**

* SPF计算

  当同区域所有路由器的链路状态数据库LSDB同步只会，每台服务器以自己为根计算出到达每个网络的最优路径，图中假设所有链路的代价均相同

  ![image-20190907194320435](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907194320435.png)



# OSPF报文

![image-20190907194454876](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907194454876.png)

![image-20190907194513608](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907194513608.png)

## OSPF报文首部结构

![image-20190907194603213](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907194603213.png)

 OSPF的**类型**如下

![image-20190907194659078](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907194659078.png)

## OSPF的五种报文工作流程

![image-20190907195110425](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907195110425.png)

## LSA首部（LSA摘要）

![image-20190907195415243](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907195415243.png)

LSA类型有如下的几种：

![image-20190907195523820](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907195523820.png)

# OSPF网络类型

![image-20190907195850745](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907195850745.png)

![image-20190907195922450](/Users/chenyansong/Documents/note/images/computeNetwork/image-20190907195922450.png)

