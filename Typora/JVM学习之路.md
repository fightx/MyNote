# 深入了解java虚拟机

## 第一章 认识java

### 1.Java技术体系

1. ​	Java程序设计语言
2. ​    Java虚拟机
3.    Class文件格式
4. ​    Java API 类库
5. ​    商业机构，开源社区的第三方Java 类库

### 2.sdk、jdk、jre的区别

1. SDK ( Software Development Kit  )

   ​	一般是软件工程为特定的软件包、软件框架、硬件平台、操作系统等建立软件应用时的开发工具的集合。

2. JDK ( Java Development Kit )

   ​	java程序设计语言、java虚拟机、java API类库这三部分成为JDK。JDK是支持java程序开发的最小环境

3. JRE  ( Java Runtime Environment )

   ​	java API类库中的Java SE API子集、Java虚拟机成为JRE。JRE是支持Java运行环境的标准环境

   ​	所以，普通用户在访问某些网站的时候，提示需要安装java运行环境只需要安装jre即可。而相关的开发人员想要进行java开发的话，就需要下载jdk。

4. jdk的开发版本号和jdk的发行版本号。自从jdk1.3以来主版本发布都会使用动物命名，而修正版本则以昆虫命名。jdk1.5,以后，公开的发行版本为 jdk 5，jdk 6...而对应的开发版本号为jdk 1.5，jdk 1.6 。所以当别人聊jdk 6的时候，你应该知道了他说的其实是jdk的发行版本号。

5. ![](C:\Users\罗蒙\AppData\Roaming\Typora\typora-user-images\image-20210411235502491.png)

### 3.Sun Hot Spot VM

1. 准确式内存管理 ( Exact Memory Management )

   虚拟机可以知道某个位置的数据具体是什么类型，譬如内存中有一个32位的整数123456，他到底是一个reference类型指向123456的内存地址，还是一个数值为123456的整数，虚拟机能分辨出来。

   这样才能在GC的时候准确判断堆上的数据是否还可能被使用，抛弃了以前的classic VM 使用的基于handler的对象查找方式，定位对象的少了一次间接的开销，提高了性能

2. 热点代码探测能力

### 4.JDK历史

1. jdk 1.0 。技术包括：java虚拟机、applet、awt 。
2. jdk1.1.内部类，反射、jar文件格式、jdbc、javabeans、rmi
3. jdk 1.2。拆分技术体系，分别为j2se（桌面级应用）,j2ee（企业级应用，crm）,j2me（移动应用） 。技术：EJB、 Java Plugin- in、Java IDL、Swing。第一次内置了JIT编译器。
4. jdk 1.3 类库改进。
5. jdk 1.4 。Java走向成熟的一个版本。新的技术特性：正则表达式、异常链、NIO、日志类、XML解析器、XSLT转换器等。
6. jdk 1.5 。语法上巨大改进，自动装箱、泛型、foreach、动态注解、枚举、可变长参数、遍历循环等。改进了内存模型。提供了java.util.concurrent 并发包。
7. jdk 1.6 。对虚拟机内部做了大量改进。
8. .jdk 1.7 。提供新的G1收集器、加强对非java语言的调用、语言级的模块化支持、升级类加载架构等。
9. .jdk 1.8 。新增lambda 表达式、提供函数式接口等。




