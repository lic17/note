---
title: Spark2.0新特性之whole-stage code generation技术和vectorization技术
categories: spark  
tags: [spark]
---


# Whole-stage code generation


之前讲解了手工编写的代码的性能，为什么比Volcano Iterator Model要好。所以如果要对Spark进行性能优化，一个思路就是在运行时动态生成代码，以避免使用Volcano模型，转而使用性能更高的代码方式。要实现上述目的，就引出了Spark第二代Tungsten引擎的新技术，whole-stage code generation。通过该技术，SQL语句编译后的operator-treee中，每个operator执行时就不是自己来执行逻辑了，而是通过whole-stage code generation技术，动态生成代码，生成的代码中会尽量将所有的操作打包到一个函数中，然后再执行动态生成的代码。


<!--more-->

就以上一讲的SQL语句来作为示例，Spark会自动生成以下代码。如果只是一个简单的查询，那么Spark会尽可能就生成一个stage，并且将所有操作打包到一起。但是如果是复杂的操作，就可能会生成多个stage。


![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/Whole-stage_code_generation.png)



Spark提供了explain()方法来查看一个SQL的执行计划，而且这里面是可以看到通过whole-stage code generation生成的代码的执行计划的。如果看到一个步骤前面有个*符号，那么就代表这个步骤是通过该技术自动生成的。在这个例子中，Range、Filter和Aggregation都是自动生成的，Exchange不是自动生成的，因为这是一个网络传输数据的过程。

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/Whole-stage_code_generation2.png)


很多用户会疑惑，从Spark 1.1版本开始，就一直听说有code generation类的feature引入，这跟spark 2.0中的这个技术有什么不同呢。实际上在spark 1.x版本中，code generation技术仅仅被使用在了expression evoluation方面（比如a + 1），即表达式求值，还有极其少数几个算子上（比如filter等）。而spark 2.0中的whole-stage code generation技术是应用在整个spark运行流程上的。


# Vectorization(向量化)
对于很多查询操作，whole-stage code generation技术都可以很好地优化其性能。但是有一些特殊的操作，却无法很好的使用该技术，比如说比较复杂一些操作，如parquet文件扫描、csv文件解析等，或者是跟其他第三方技术进行整合。

如果要在上述场景提升性能，spark引入了另外一种技术，称作“vectorization”，即向量化。向量化的意思就是避免每次仅仅处理一条数据，相反，将多条数据通过面向列的方式来组织成一个一个的batch，然后对一个batch中的数据来迭代处理。每次next()函数调用都返回一个batch的数据，这样**可以减少virtual function dispatch的开销。同时通过循环的方式来处理，也可以使用编译器和CPU的loop unrolling等优化特性。**

![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/Whole-stage_code_generation3.png)



**这种向量化的技术，可以使用到之前说的3个点中的2个点。即，减少virtual function dispatch，以及进行loop unrolling优化**。但是还是需要通过内存缓冲来读写中间数据的(无法使用cpu register读取数据)。所以，仅仅当实在无法使用whole-stage code generation时，才会使用vectorization技术。有人做了一个parquet文件读取的实验，采用普通方式以及向量化方式，性能也能够达到一个数量级的提升：


![](http://ols7leonh.bkt.clouddn.com//assert/img/bigdata/spark从入门到精通_笔记/Whole-stage_code_generation4.png)

上述的whole-stage code generation技术，能否保证将spark 2.x的性能比spark 1.x来说提升10倍以上呢？这是无法完全保证的。虽然说目前的spark架构已经搭载了目前世界上最先进的性能优化技术，但是并不是所有的操作都可以大幅度提升性能的。简单来说，CPU密集型的操作，可以通过这些新技术得到性能的大幅度提升，但是很多IO密集型的操作，比如shuffle过程的读写磁盘，是无法通过该技术提升性能的。在未来，spark会花费更多的精力在优化IO密集型的操作的性能上。





