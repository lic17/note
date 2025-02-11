---
title: shell中各种括号的作用()、(())、[]、[[]]、{}
categories: shell   
toc: true  
tags: [shell]
---



[TOC]




# 1.()

## 1.1.命令组
&emsp;括号中的命令将会新开一个子shell顺序执行，所以括号中的变量不能够被脚本余下的部分使用。括号中多个命令之间用分号隔开，最后一个命令可以没有分号，各命令和括号之间不必有空格。


## 1.2.用于初始化数组
```
array=(a b c d)
```

# 2.(())

## 2.1.四则运算
```
 a=5; ((a++)) 可将 $a 重定义为6
```


## 2.2.逻辑运算
```
直接使用for((i=0;i<5;i++))
```

# 3.[]

## 3.1.test测试

详见：条件测试及控制流.docx

## 3.2.数组下标

在一个array 结构的上下文中，中括号用来引用数组中每个元素的编号

# 4[[]]

使用[[ ... ]]条件判断结构

直接使用if [[ \$a != 1 && \$a != 2 ]], 如果不适用双括号, 则为if [ \$a -ne 1] && [ \$a != 2 ]或者if [ \$a -ne 1 -a \$a != 2 ]。

# 5.{}

## 5.1对大括号中的以逗号分割的文件列表进行拓展
```

touch {a,b}.txt
结果为：a.txt     b.txt
```

## 5.2对大括号中以点点（..）分割的顺序文件列表起拓展作用

```
如：touch {a..d}.txt
结果为a.txt b.txt c.txt d.txt
```


## 5.3.代码块
&emsp;又被称为内部组，这个结构事实上创建了一个匿名函数 。与小括号中的命令不同，大括号内的命令不会新开一个子shell运行，即脚本余下部分仍可使用括号内变量。括号内的命令间用分号隔开，最后一个也必须有分号。{}的第一个命令和左括号之间必须要有一个空格

## 5.4其他

```
类似于下面的结构，对字符串进行操作，详细见：shell.docx 中“字符串处理函数”
${var:-DEFAULT}

```



# && 与  ||

| 命令          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| cmd1&&cmd2    | 若 cmd1 执行完毕且正确执行（$?=0） ，则开始执行 cmd2。 <br>  若 cmd1 执行完毕且为错误 （$?≠0） ，则 cmd2 不执行。 |
| cmd1 \|\|cmd2 | 若 cmd1 执行完毕且正确执行（$?=0） ，则 cmd2 不执行。<br>若 cmd1 执行完毕且为错误 （$?≠0） ，则开始执行 cmd2。 |





