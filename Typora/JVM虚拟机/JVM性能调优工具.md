# 说明

现实企业级Java开发中，会碰到如下问题：
1.OutOfMemoryError，内存不足
2.内存溢出
3.线程锁死
4.锁征用
5.Java进程消耗CPU过高
通过JVM一些常用的监控性能调优工具，可以很好的监控与发现问题所在。

在中银项目中出现了内存溢出的现象，通过使用各种工具配合使用来发现是那些对象没有成功释放，从而解决内存溢出的问题。





















# jps

jps（Java Virtual Machine Process Status Tool）主要用来输出JVM中运行的进程状态信息。语法格式如下：

```
jps [options] [hostid]
```

如果不指定hostid就默认为当前主机或服务器。
命令行参数如下：

```
-q 不输出类名、Jar名和传入main方法的参数
-m 输出传入main方法的参数
-l 输出main类或Jar的全限名
-v 输出传入JVM的参数
```

如下图：
![img](https://img.kancloud.cn/45/2a/452a9a288d9dcf43fa7260ca48ecc645_907x331.png)











# jstack

jstack主要用来查看某个Java进程内的线程堆栈信息。语法格式如下：

```
jstack [option] pid
jstack [option] executablecore
jstack [option] [server-id@]remote-hostname-or-ip
```

命令行参数如下：

```
-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）
```

jstack可以定位到线程堆栈，根据堆栈信息我们可以定位到具体代码，所以它在JVM性能调优中使用得非常多。找出某个Java进程中最耗费CPU的Java线程并定位堆栈信息，用到的命令有ps、top、printf、jstack、grep。操作如下：
1.查询Java进程ID，如下图：
![img](https://img.kancloud.cn/80/88/80880ea353d29bc2af639dacac7e54e6_1253x250.png)
2.通过进程ID查询对CPU消耗最大的线程，可以使用ps -Lfp pid或者ps -mp pid -o THREAD，tid，time或者top -Hp pid，我这里用第三个，如下图：
![img](https://img.kancloud.cn/c8/51/c8514733a77455ac5cda422b8c50a66c_792x392.png)
将线程ID转成16进制用于查询，使用如下命令即可实现：

```
printf "%x\n" pid
```

3.使用jstack输出进程的堆栈信息，如下图：
![img](https://img.kancloud.cn/b3/cd/b3cd0af7a852133daee449cb6ecacd2e_763x263.png)

















# jmap和jhat

jmap（Memory Map）用来查看堆内存使用状况，一般结合jhat（Java Heap Analysis Tool）使用。语法格式如下：

```
jmap [option] pid
jmap [option] executablecore
jmap [option] [server-id@]remote-hostname-or-ip
```

1.没有参数打印进程的类加载器和类加载器加载的持久代对象信息，输出：类加载器名称、对象是否存活（不可靠）、对象地址、父类加载器、已加载的类大小等信息，如下图：

```
jmap pid
```

![img](https://img.kancloud.cn/fa/64/fa64afebe2bedd82e3093ba3c8791f58_1920x784.png)
2.查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况。比如下面的例子：

```
jmap -heap pid
```

![img](https://img.kancloud.cn/e4/91/e491c0650bcd0646646ac40bd198ac49_1920x784.png)
3.使用jmap -histo[:live] pid查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象并且会强制执行一次GC，如下：

```
jmap -histo pid
jmap -histo:live pid
```

![img](https://img.kancloud.cn/da/83/da83ad60e4c012a7b9926e9772fc0cb7_1920x784.png)
![img](https://img.kancloud.cn/2e/f1/2ef1a35d8d3fd445496543959e6c4f34_1920x784.png)
class name是对象类型，说明如下：

```
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
```

还有一个很常用的情况是：用jmap把进程内存使用情况dump到文件中，再用jhat分析查看。jmap进行dump命令格式如下：

```
jmap -dump:format=b,file=dumpFileName pid (文件的后缀建议采用".hprof")
```

![img](https://img.kancloud.cn/64/88/648865b43b9f8ef7edcd41081143066b_1920x784.png)
dump出来的文件可以用MAT、VisualVM等工具查看，这里用jhat查看：

```
jhat -port xxxx dumpFileName
```

![img](https://img.kancloud.cn/3b/46/3b460b42b2b1a1cf3ccc0d6207b2b543_1920x784.png)
然后就可以在浏览器中输入主机地址:port查看了：
![img](https://img.kancloud.cn/f3/15/f315dab16ed2f08552d7016e18fb7a23_1920x1030.png)



















# jstat

jstat（JVM统计监测工具），监控的内容有：类装载、内存、垃圾收集、jit编译的信息。语法格式如下：

```
jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]
```

vmid是虚拟机ID，在Linux/Unix系统上一般就是进程ID。interval是采样时间间隔。count是采样数目。比如下面输出的是GC信息，采样时间间隔为250ms，采样数为4：
![img](https://img.kancloud.cn/fc/b8/fcb84478358ae88db799705d2d2b0d26_1920x784.png)
要明白上面各列的意义，先看JVM堆内存布局：
![img](https://img.kancloud.cn/9f/fc/9ffc9bfcf35c17cef29284415e0a6171_300x158.png)
可以看出：

```
堆内存 = 年轻代 + 年老代 + 永久代
年轻代 = Eden区 + 两个Survivor区（From和To）
```

各列含义：

```
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
PC、PU：永久代容量和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时
```





























# jinfo

jinfo（实时查看与调整虚拟机的各项参数），语法格式如下：

```
jinfo [option] <pid>
jinfo [option] <executable <core>
jinfo [option] [server_id@]<remote server IP or hostname>
```



















# jconsole

jconsole，作用有：内存监控、线程监控、死锁。