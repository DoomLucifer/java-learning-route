[TOC]

# JavaCore

## Java平台概论

### 基础介绍

#### Java SE

1. JVM（Java Virtual Machine），包括在JRE中
2. JRE（Java SE Runtime Environment），要运行java程序，必须安装JRE
3. JDK（Java SE Development Kids），包括JRE及开发过程中需要的一些工具程序，像javac、java、appletviewer等工具程序。要开发java程序，必须安装JDK

#### Java EE

以Java SE为基础，定义了一系列的服务、API、协议等，比较为人熟悉的技术像JSP、Servlet、JavaMail、Enterprise JavaBeans（EJB）等

#### Java ME

#### JCP&JSR

JCP（Java Community Process）

JSR（Java Specification Requests）

任何想要提议加入Java的功能或特性，必须以JSR正式文件的方式提交，JSR必须经过JCP执行委员会投票通过，方可成为最终标准文件。

查询特性版本的规范：

http://jcp.org/en/jsr/detail?id=版本号

比如查询JSR 337文件规范：

http://jcp.org/en/jsr/detail?id=337

#### Java Learning Route

1. 深入了解JVM/JRE/JDK
2. 理解封装、继承、多态
3. 掌握常用Java SE API架构

掌握常用的Java SE API，如异常（Exception）、集合（Collection）、输入/输出流（Stream）、线程（Thread）等。学习这些标准的API，不要死背，应先掌握API在设计时的封装、继承、多态架构。以Collection为例，在学习时必须先理解为何要设计下图的架构

![image-20181026204510707](/Users/zhengguoqiang/Library/Application Support/typora-user-images/image-20181026204510707.png)

4. 学习容器观念
5. 研究开源项目
6. 学习设计模式与重构
7. 熟悉相关开发工具

### JVM

JVM可以让Java跨平台

那么什么是跨平台，不跨平台又是怎么一回事？

对于计算机而言，只识别一种语言，就是0、1序列组成的机器指令。

![image-20181031135810011](/var/folders/hc/8pfk3ys17jsd7bl2t0tjs6x80000gn/T/abnerworks.Typora/image-20181031135810011.png)

问题在于不同平台识别的0、1序列不一样，因此必须使用不同的编译程序为不同的平台编译出可执行的机器码，在Windows平台编译好的程序，无法直接拿到其他平台运行，即无法达到“编译一次，到处运行”的跨平台的目的。

![image-20181031143233287](/var/folders/hc/8pfk3ys17jsd7bl2t0tjs6x80000gn/T/abnerworks.Typora/image-20181031143233287.png)

