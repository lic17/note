---
title: zookeeper安装配置(单机，伪分布式、集群)
categories: hadoop   
tags: [zookeeper]
---

[TOC]





# 1.单机模式搭建

## 1.1.解压
```
cd /home/oldboy/tools/                
tar -zxvf zookeeper-3.4.5.tar.gz 
mv zookeeper-3.4.5 zookeeper

```

## 1.2.修改配置文件
```
cd zookeeper/conf/ 
cp   zoo_sample.cfg   zoo_sample.cfg.bak          #首先备份配置文件
mv   zoo_sample.cfg   zoo.cfg               #将配置文件改名


'修改下列选项'
tickTime=2000
dataDir=/usr/local/zk/data
dataLogDir=/usr/local/zk/log      
clientPort=2181

```

## 1.3.配置环境变量
为了今后操作方便，我们需要对Zookeeper的环境变量进行配置，方法如下在/etc/profile文件中加入如下内容：

```
'需要Java的环境，所以如果没有jdk，需要配置jdk'
ZOOKEEPER=/home/oldboy/tools/zookeeper
PATH=$ZOOKEEPER/bin:$PATH
export PATH
```

## 1.4.启动/停止服务
```
#启动ZooKeeper的Server
./bin/zkServer.sh start

#关闭ZooKeeper的Server
./bin/zkServer.sh stop

```



# 2.伪集群模式搭建
## 2.1.复制及修改配置文件
```
[root@data-1-1 ~]# cd /home/oldboy/tools/zookeeper
[root@data-1-1 zookeeper0]# cp ./conf/zoo.cfg zoo1.cfg
[root@data-1-1 zookeeper0]# cp ./conf/zoo.cfg zoo2.cfg
[root@data-1-1 zookeeper0]# cp ./conf/zoo.cfg zoo3.cfg

'##################   zoo1.cfg   ###########################'
# The number of milliseconds of each tick
tickTime=2000
 
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
 
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
 
# the directory where the snapshot is stored.
dataDir=/usr/local/zk/data_1
 
# the port at which the clients will connect
clientPort=2181
 
#the location of the log file
dataLogDir=/usr/local/zk/logs_1
 
server.0=localhost:2287:3387
server.1=localhost:2288:3388
server.2=localhost:2289:3389

'##################   zoo2.cfg   ###########################'
# The number of milliseconds of each tick
tickTime=2000
 
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
 
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
 
# the directory where the snapshot is stored.
dataDir=/usr/local/zk/data_2
 
# the port at which the clients will connect
clientPort=2182
 
#the location of the log file
dataLogDir=/usr/local/zk/logs_2
 
server.0=localhost:2287:3387
server.1=localhost:2288:3388
server.2=localhost:2289:3389

'##################   zoo3.cfg   ###########################'
# The number of milliseconds of each tick
tickTime=2000
 
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
 
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
 
# the directory where the snapshot is stored.
dataDir=/usr/local/zk/data_3
 
# the port at which the clients will connect
clientPort=2183
 
#the location of the log file
dataLogDir=/usr/local/zk/logs_3
 
server.0=localhost:2287:3387
server.1=localhost:2288:3388
server.2=localhost:2289:3389


'#其实上述配置文件就是：clientPort、dataDir、dataLogDir、server.x 不同'
```
## 2.2.创建相应的文件
```
//创建dataLogDir
mkdir /usr/local/zk/logs_1 -p
mkdir /usr/local/zk/logs_2 -p
mkdir /usr/local/zk/logs_3 -p

//创建dataDir
mkdir /usr/local/zk/data_1 -p
mkdir /usr/local/zk/data_2 -p
mkdir /usr/local/zk/data_3 -p

//在dataDir下创建myid
/*
在/usr/local/zk/data_x目录下创建myid文件，在对应的myid文件中写入数字，
server.X和myid： server.X 这个数字就是对应，data/myid中的数字。在3个server的myid文件中分别写入了0，1，2
*/
echo "0">/usr/local/zk/data_1/myid
echo "1">/usr/local/zk/data_2/myid
echo "2">/usr/local/zk/data_3/myid

cat    /usr/local/zk/data_3/myid        #检查一下

```
## 2.3.启动
```
[root@data-1-1 conf]# zkServer.sh start zoo1.cfg               
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo1.cfg        #为什么没有指定路径，因为配置了zk的环境变量，这里就可以看到回去找对应的bin和conf目录
Starting zookeeper ... STARTED

[root@data-1-1 conf]# zkServer.sh start zoo2.cfg
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo2.cfg
Starting zookeeper ... STARTED

[root@data-1-1 conf]# zkServer.sh start zoo3.cfg
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo3.cfg
Starting zookeeper ... STARTED

//检查，是3个表示启动成功
[root@data-1-1 conf]# jps
2566 QuorumPeerMain
2539 QuorumPeerMain
2638 Jps
2607 QuorumPeerMain
[root@data-1-1 conf]#


//检查2：通过命令的方式
[root@data-1-1 tools]# zkServer.sh status zoo1.cfg
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo1.cfg
Mode: follower

[root@data-1-1 tools]# zkServer.sh status zoo2.cfg
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo2.cfg
Mode: leader

[root@data-1-1 tools]# zkServer.sh status zoo3.cfg
JMX enabled by default
Using config: /home/oldboy/tools/zookeeper0/bin/../conf/zoo3.cfg
Mode: follower

 
```



# 3.集群模式搭建

## 3.1.相同的配置文件
创建一个配置文件zoo.cfg

```
# The number of milliseconds of each tick
tickTime=2000
 
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
 
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
 
# the directory where the snapshot is stored.
dataDir=/usr/local/zk/data
 
# the port at which the clients will connect
clientPort=2183
 
#the location of the log file
dataLogDir=/usr/local/zk/log
 
server.0=hadoop:2288:3388
server.1=hadoop0:2288:3388
server.2=hadoop1:2288:3388

#此时的server.x就相同，因为配置文件在不同的机器上，所以不必担心端口冲突的问题
```

## 3.2.创建myid
```
在dataDir(/usr/local/zk/data)目录创建myid文件
 
Server0机器的内容为：0
Server1机器的内容为：1
Server2机器的内容为：2
 
```
## 3.3.创建dataDir和dataLogDir
在3台机器上都执行

```
//创建dataLogDir
mkdir /usr/local/zk/logs -p

 
//创建dataDir
mkdir /usr/local/zk/data -p

```
## 3.4.启动
在3台机器上都执行

```
zkServer.sh start

//检查
zkServer.sh status

```

## 3.5.客户端连接
```
zkCli.sh -server 192.168.0.50:2182                        #带-server连接的是远端
zkCli.sh                #连接的是本地
```


# 错误集锦

```
在我们需要修改指定的目录，然后将这样注释，
#dataDir=/tmp/zookeeper

回报下面的错误：
[root@node01 app]# /home/hadoop/app/zookeeper-3.3.6/bin/zkServer.sh start
JMX enabled by default
Using config: /home/hadoop/app/zookeeper-3.3.6/bin/../conf/zoo.cfg
Starting zookeeper ... /home/hadoop/app/zookeeper-3.3.6/bin/zkServer.sh: line 93: [: /tmp/zookeeper: binary operator expected
/home/hadoop/app/zookeeper-3.3.6/bin/zkServer.sh: line 103: /tmp/zookeeper
/usr/local/zk/data/zookeeper_server.pid: No such file or directory
FAILED TO WRITE PID

有点坑

```


```
java.lang.RuntimeException: My id 4 not in the peer list
    搭建集群的时候需要配置zookeeper节点，同时需要在zoo.cfg中的文件配置服务节点。这个问题主要是因为上述两处配置没有相互对应起来，我虽然在myid中配置了节点4但是我在zoo.cfg的文件直接配置server1,2,3并没有节点4，而且zookeeper的节点必须从1开始，所以在配置文件的时候还是在myid中按顺序添加吧。

vim $zk_home/data/myid

vim $zk_home/conf/zoo.cfg 

server.1=soc62:2888:3888
server.2=soc63:2888:3888
server.3=soc64:2888:3888

```

