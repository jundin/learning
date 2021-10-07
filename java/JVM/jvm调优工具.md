
### jcmd
https://blog.csdn.net/jjs15259655776/article/details/108188972



### jconsole (Java Monitoring and Management Console)
>一个java GUI监视工具，可以以图表化的形式显示各种数据。并可通过远程连接监视远程的服务器VM。用java写的GUI程序，用来监控VM，并可监控远程的VM，非常易用，而且功能非常强。命令行里打 jconsole，选则进程就可以了。

> 看到内存、线程、类及CPU使用的一些情况。
>
> 线程页，可以检测死锁，查看死锁信息

### jvisualVM
>VisualVM 是一款免费的集成了多个JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优。这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作。

>在内存分析上，Java VisualVM的最大好处是可通过安装Visual GC插件来分析GC（Gabage Collection）趋势、内存消耗详细状况。

Visual GC安装插件完成后重启Java VisualVM，Visual GC界面自动打开，即可看到JVM中堆内存的分代情况

检测到死锁！
生成一个线程 Dump 以获取更多信息


### Jstat
>用于监控基于HotSpot的JVM，对其堆的使用情况进行实时的命令行的统计，使用jstat我们可以对指定的JVM做如下监控：

- 类的加载及卸载情况

- 查看新生代、老生代及持久代的容量及使用情况

- 查看新生代、老生代及持久代的垃圾收集情况，包括垃圾回收的次数及垃圾回收所占用的时间

- 查看新生代中Eden区及Survior区中容量及分配情况等

jstat工具特别强大，它有众多的可选项，通过提供多种不同的监控维度，使我们可以从不同的维度来了解到当前JVM堆的使用情况。详细查看堆内各个部分的使用量，使用的时候必须加上待统计的Java进程号，可选的不同维度参数以及可选的统计频率参数。