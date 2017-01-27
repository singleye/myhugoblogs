+++
date = "2017-01-27T17:10:07+08:00"
title = "玩转阿波罗11号飞船导航计算机模拟器"
keywords = ["Apollo", "AGC", "Saturn", "Command Module", "Lunar Module"]
summary = "前两天在Github上看到一个有趣的项目，是当年美国登月计划“阿波罗11号”的导航计算机的源代码，并且Virtual AGC和MIT museum还开发了运行用的模拟器，使用这个模拟器能够运行导航计算机中的“指令模块”和“登月模块”。"
categories = ["Technology", "Hacking"]
tags = ["Apollo", "AGC", "Lunar"]
+++

![土星号](http://www.ibiblio.org/apollo/Saturn1B-transparent.png)

PS： 上个周末还在听王冠红人馆调侃河南空气质量监督特别小组的禁放政策出台3天后就撤销的节目，心想今年过年回家乡鞭炮还可以照常放，没想到还真的全面禁放了。没办法放鞭炮咱就“发射个火箭”玩玩吧:-)

前两天在Github上看到一个有趣的项目，是当年美国登月计划[“阿波罗号”](https://github.com/virtualagc/virtualagc)的导航计算机的源代码，并且[Virtual AGC](http://www.ibiblio.org/apollo/) [MIT museum](http://mitmuseum.mit.edu)还开发了运行用的模拟器，使用这个模拟器能够运行导航计算机中的“指令模块”和“登月模块”。

估计很多人看过下面的张照片。没错，那等身的汇编代码现在可以在这个项目里面看到了！
![Margaret Hamilton](http://news.mit.edu/sites/mit.edu.newsoffice/files/styles/news_article_image_top_slideshow/public/images/2016/margaret-hamilton-mit-apollo-code.jpeg?itok=Gd6FBFbH)

那么这个程序究竟是干什么的呢？要知道飞往月球并返回地球依靠宇航员手动是不可能的事情，当然也没有什么GPS，飞船必须沿着预定轨道航行才可以，这个程序就是保证飞船正确航行的导航程序。MIT仪器实验室为阿波罗计划开发了这套导航系统，这套导航计算机也叫“AGC (Apollo Guidance Computer)”，飞船上有有两套这样的导航系统，分别运行了不同的程序，一个是CM (Command Module)用来把宇航员送往月球并且返回，另一个是LM (Lunar Module)用来把宇航员送到月球表面并且返回飞船。

先来看看这货到底长什么样子吧。PS: 体格上好像当年玩过的EMC 60块盘的DAE...
![AGC](http://www.ibiblio.org/apollo/P6210643_AGC.png)

* 2048字（不到4kB）的内存
* 36,864字（不到72kB）的只读存储
* 每秒钟最多可执行85,000条指令
* 尺寸: 24"×12.5"×6" (英寸).
* 重达70.1磅
* 使用28V 2.5A的直流电

显示器和键盘(DSKY)看起来就跟一个普通计算器差不多。
![DSKY](http://www.ibiblio.org/apollo/RealDSKY-thumb.jpg)


故事先讲到这儿，来看看怎样把这个模拟器跑起来吧。
## 1. 下载代码库
{{< highlight shell "linenos=inline,style=manni" >}}
$ git clone https://github.com/virtualagc/virtualagc.git
{{< /highlight >}}

## 2. 编译代码
我使用的环境是CentOS 6.5，为了编译通过需要对Makefile做一些小的修改：
{{< highlight shell "linenos=inline,style=manni" >}}
$ cd virtualagc
$ git diff Makefile yaAGCb1/Makefile
diff --git a/Makefile b/Makefile
index 773edf2..0e9ff8c 100644
--- a/Makefile
+++ b/Makefile
@@ -321,7 +321,7 @@ LIBS+=-lwsock32
 CURSES=../yaAGC/random.c
 CURSES+=-lregex
 else
-CURSES=-lcurses
+CURSES=-lncurses
 endif

 ifdef MACOSX
diff --git a/yaAGCb1/Makefile b/yaAGCb1/Makefile
index faf4c5e..e779b0b 100644
--- a/yaAGCb1/Makefile
+++ b/yaAGCb1/Makefile
@@ -24,7 +24,7 @@ endif
 all default: ${TARGETS}

 yaAGCb1: $(SOURCE) $(HEADERS) Makefile
-       ${cc} ${CFLAGS0} -O0 -g -o $@ $(SOURCE) -lpthread $(LIBS)
+       ${cc} ${CFLAGS0} -O0 -g -o $@ $(SOURCE) -lpthread -lrt $(LIBS)

 test.agc.bin: test.agc
        ../../virtualagc/yaYUL/yaYUL --block1 $^ >test.agc.lst
{{< /highlight >}}

编译
{{< highlight shell "linenos=inline,style=manni" >}}
$ make
$ make install
{{< /highlight >}}
完成后会在用户目录下出现"VirtualAGC"目录，可运行程序都被安装到这个目录下。


## 3. 飞向月球吧
运行bin目录下的VirtualAGC即可运行起来。
{{< highlight shell "linenos=inline,style=manni" >}}
$ ./bin/VirtualAGC
{{< /highlight >}}
![VirtualAGC](http://imglf2.nosdn.127.net/img/NGpJbGd3MUdOUDN1eHVZY2NuYjJrM1dQVjlHOWwyRk1wQ3k0eTVocm9zY0VCNDIvMThmbWFRPT0.gif)

列几个CM的命令：

| 命令 | 功能 |
| ---- | ---- |
| V05N09E | 查看报警代码 |
| V35E | DSKY灯管测试 |
| V91E V33E | 查看内存bank检矫码，使用V33E指令来选择其他内存bank |
| V16N36E/V16N65E | 查看系统当前时间 |
| V25N36E | 设置时间 |
| V27N02E | 查看core-rope中的内容，通过键盘在R3中输入8进制的地址来查看其中的内容 |
| V01N02E | 查看可擦写内存中的内容 |
| V21N02E | 修改内存的内容 |
| V36E | 重启 |

完整的命令列表可以在Github上的代码中[ASSEMBLY_AND_OPERATION_INFORMATION.s](https://github.com/virtualagc/virtualagc/blob/master/Colossus249/ASSEMBLY_AND_OPERATION_INFORMATION.agc)看到。

几个LM的命令：

| 命令 | 功能 |
| ---- | ---- |
| V35E | DSKY灯管测试 |
| V37E 00E | 启动P00号程序 |
| V25E N01E 01365E 0E 0E 0E | 设置自检测试失败次数 |
| V21 N27E 10E | 启动自检程序 |
| V21 N27E 0E | 关闭自检程序 |

完整的命令列表可以在Github上的代码中[ASSEMBLY_AND_OPERATION_INFORMATION.s](https://github.com/virtualagc/virtualagc/blob/master/Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc)看到。
