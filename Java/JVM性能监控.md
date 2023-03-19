# 性能监控和调优

## 概述

### 背景说明
**生产环境中的问题**
- 生产环境发生了内存溢出该如何处理？
- 生产环境应该给服务器分配多少内存合适？
- 如何对垃圾回收器的性能进行调优？
- 生产环境CPU负载飙高该如何处理？
- 生产环境应该给应用分配多少线程合适？
- 不加log，如何确定请求是否执行了某一行代码？
- 不加log，如何实时查看某个方法的入参与返回值？

**为什么要调优**  
- 防止出现OOM
- 解决OOM
- 减少Full GC出现的频率

**不同阶段的考虑**  
- 上线前
- 项目运行阶段
- 线上出现OOM

### 调优概述

**监控的依据**  
- 运行日志
- 异常堆栈
- GC日志
- 线程快照
- 堆转储快照

**调优的大方向**   
- 合理地编写代码
- 充分并合理的使用硬件资源
- 合理地进行JVM调优

### 性能优化的步骤  
**第1步：性能监控**  
- GC频繁
- cpu load过高
- OOM
- 内存泄露
- 死锁
- 程序响应时间较长

**第2步：性能分析**  
- 打印GC日志，通过GCviewer或者 http://gceasy.io 来分析异常信息
- 灵活运用命令行工具、jstack、jmap、jinfo等
- dump出堆文件，使用内存分析工具分析文件
- 使用阿里Arthas、jconsole、JVisualVM来实时查看JVM状态
- jstack查看堆栈信息

**第3步：性能调优**  
- 适当增加内存，根据业务背景选择垃圾回收器
- 优化代码，控制内存使用
- 增加机器，分散节点压力
- 合理设置线程池线程数量
- 使用中间件提高程序效率，比如缓存、消息队列等
- 其他……

### 性能评价/测试指标
停顿时间（或响应时间）

提交请求和返回该请求的响应之间使用的时间，一般比较关注平均响应时间。常用操作的响应时间列表：

操作 | 响应时间|
---------|----------|
打开一个站点 |	几秒|
数据库查询一条记录（有索引） |	十几毫秒|
机械磁盘一次寻址定位 |	4毫秒|
从机械磁盘顺序读取1M数据 |	2毫秒|
从SSD磁盘顺序读取1M数据 |	0.3毫秒|
从远程分布式换成Redis 读取一个数据 |	0.5毫秒|
从内存读取 1M数据 |	十几微秒|
Java程序本地方法调用 |	几微秒|
网络传输2Kb数据 |	1 微秒|

**在垃圾回收环节中：**  
- 暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间。
- `-XX:MaxGCPauseMillis`

**吞吐量**  
- 对单位时间内完成的工作量（请求）的量度
- 在GC中：运行用户代码的事件占总运行时间的比例（总运行时间：程序的运行时间+内存回收的时间）
- 吞吐量为1-1/(1+n)，其中`-XX::GCTimeRatio=n`

**并发数**  
- 同一时刻，对服务器有实际交互的请求数(1000人在线，并发数在5%-15%)

**内存占用**  
- Java堆区所占的内存大小

**相互间的关系**

**以高速公路通行状况为例**   
- 吞吐量：每天通过高速公路收费站的车辆的数据
- 并发数：高速公路上正在行驶的车辆的数目
- 响应时间：车速




## 命令行工具
在JDK安装目录中 `bin`目录下有官方提供的`javap` `jcmd` `jconsole` `jhat` `jinfo` `jmap` `jmc` `jps` `jstack`等相关工具   

`bin`目录下的工具实际代码位置在`lib/tool.jar`中，如图：   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/自带命令行工具代码位置.png)

### jps
查看正在运行的Java进程
jps(Java Process Status)：显示指定系统内所有的HotSpot虚拟机进程（查看虚拟机进程信息），可用于查询正在运行的虚拟机进程。

说明：对于本地虚拟机进程来说，进程的本地虚拟机ID与操作系统的进程ID是一致的，是唯一的。

基本使用语法为：`jps [options] [hostid]`   
`hostid`参数用于远程连接   

我们还可以通过追加参数，来打印额外的信息。

**options参数**   
- `-q`：仅仅显示LVMID（local virtual machine id），即本地虚拟机唯一id。不显示主类的名称等
- `-l`：输出应用程序主类的全类名 或 如果进程执行的是jar包，则输出jar完整路径
- `-m`：输出虚拟机进程启动时传递给主类main()的参数 
- `-v`：列出虚拟机进程启动时的JVM参数。比如：-Xms20m -Xmx50m是启动程序指定的jvm参数。
说明：以上参数可以综合使用。

```powershell
PS C:\Users\hasee> jps -l -m
16992 sun.tools.jps.Jps -l -m
18356 com.jvm.GClogTest test11
```

?> 如果某 Java 进程关闭了默认开启的`UsePerfData`参数（即使用参数`-XX：-UsePerfData`），那么jps命令（以及下面介绍的jstat）将无法探知该Java 进程。



### jstat 
查看JVM的统计信息  
[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)


jstat`JVM Statistics Monitoring Tool`：用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的**类装载、内存、垃圾收集、JIT编译**等运行数据。在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是**运行期定位虚拟机性能问题的首选工具。常用于检测垃圾回收问题以及内存泄漏问题**。

基本使用语法为：`jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`   
查看命令相关参数：`jstat -h` 或 `jstat -help`  
其中vmid是进程id号，也就是jps之后看到的前面的号码    

#### 参数说明
- `option参数`：     
  - 类装载相关的：   
    - `-class`：显示ClassLoader的相关信息：类的装载、卸载数量、总空间、类装载所消耗的时间等
  - 垃圾回收相关的： 
    - `-gc`：显示与GC相关的堆信息。包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息。
    - `-gccapacity`：显示内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间。
    - `-gcutil`：显示内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比。
    - `-gccause`：与-gcutil功能一样，但是会额外输出导致最后一次或当前正在发生的GC产生的原因。
    - `-gcnew`：显示新生代GC状况
    - `-gcnewcapacity`：显示内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间
    - `-geold`：显示老年代GC状况
    - `-gcoldcapacity`：显示内容与-gcold基本相同，输出主要关注使用到的最大、最小空间
    - `-gcpermcapacity`：显示永久代使用到的最大、最小空间。
  - JIT相关的：
    - `-compiler`：显示JIT编译器编译过的方法、耗时等信息
    - `-printcompilation`：输出已经被JIT编译的方法
- `-t参数`： 可以在输出信息前加上一个Timestamp列，显示程序的运行时间。单位：秒   
  - ` jstat -class -t 18356`
- `-h参数`： 可以在周期性数据输出时，输出多少行数据后输出一个表头信息   
  - ` jstat -class -h3 18356`每三条信息打印一次表头
- `interval参数`： 用于指定输出统计数据的周期，单位为毫秒。即：查询间隔
  - `jstat -class 18356 1000`每秒打印一次ClassLoader相关信息    
- `count参数`： 用于指定查询的总次数 
  - `jstat -class 18356 1000 5`每秒打印一次ClassLoader相关信息，打印五次    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jstat主要参数示例.png)



#### 表头说明
表头  | 含义（字节）
---------|----------|
EC |  Eden区的大小
EU | Eden区已使用的大小
S0C | 幸存者0区的大小
S1C | 幸存者1区的大小
S0U | 幸存者0区已使用的大小
S1U | 幸存者1区已使用的大小
MC | 元空间的大小
MU | 元空间已使用的大小
OC | 老年代的大小
OU | 老年代已使用的大小
CCSC | 压缩类空间的大小
CCSU | 压缩类空间已使用的大小
YGC | 从应用程序启动到采样时young gc的次数
YGCT | 从应用程序启动到采样时young gc消耗时间（秒）
FGC | 从应用程序启动到采样时full gc的次数
FGCT | 从应用程序启动到采样时的full gc的消耗时间（秒）
GCT | 从应用程序启动到采样时gc的总时间
TT | 晋升老年代阈值
MTT | 晋升老年代最大阈值

注：详细全面的表头查看官方文档     

### jinfo
**实时查看和修改JVM配置参数**     
jinfo`Configuration Info for Java`：查看虚拟机配置参数信息，也可用于调整虚拟机的配置参数。在很多情况卡，Java应用程序不会指定所有的Java虚拟机参数。而此时，开发人员可能不知道某一个具体的Java虚拟机参数的默认值。在这种情况下，可能需要通过查找文档获取某个参数的默认值。这个查找过程可能是非常艰难的。但有了jinfo工具，开发人员可以很方便地找到Java虚拟机参数的当前值。

基本使用语法为：`jinfo [options] pid`

选项	| 选项说明 |案例
---------|----------|----|
`no option pid` |	输出全部的参数和系统属性 |`jinfo 9564`
`-flag name pid` |	输出对应名称的参数| `jinfo -flag UseG1GC 9564`
`-flag [+-]name pid` |	开启或者关闭对应名称的参数 只有被标记为`manageable的参数`才可以被动态修改 |`jinfo -flag +PrintGCDetails 25592`
`-flag name=value pid` |	设定对应名称的参数 |`jinfo -flag -UseG1GC 9564`
`-flags pid` |	输出全部的参数 |`jinfo -flags 9564`
`-sysprops pid` |	输出系统属性 |`jinfo -sysprops 9564`

> `manageable的参数` 可以通过命令查看`java -XX:+PrintFlagsFinal -version | grep manageable`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/manageable参数示例.png)  

**拓展命令**
- `java -XX:+PrintFlagsInitial` 查看所有JVM参数启动的初始值
- `java -XX:+PrintFlagsFinal` 查看所有JVM参数的最终值
- `java -XX:+PrintCommandLineFlags` 查看被用户或者JVM设置过的详细参数名和参数值


### jmap
**导出内存映像文件&内存使用情况**

jmap`JVM Memory Map`：作用一方面是获取dump文件（堆转储快照文件，二进制文件），它还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况、堆中对象的统计信息、类加载信息等。开发人员可以在控制台中输入命令“jmap -help”查阅jmap工具的具体使用方式和一些标准选项配置。

[官方文档](https://docs.oracle.com/en/java/javase/11/tools/jmap.html)

基本使用语法为：  
- `jmap [option] <pid>`
- `jmap [option] <executable <core>`
- `jmap [option] [server_id@] <remote server IP or hostname>`

**option参数说明**

- `-dump`	生成dump文件（Java堆转储快照），
  - `-dump:live`只保存堆中的存活对象
- `-heap`	输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用信息等
- `-histo`	输出堆空间中对象的统计信息，包括类、实例数量和合计容量，
  - `-histo:live`只统计堆中的存活对象
- `-J <flag>`	传递参数给jmap启动的jvm
- `-finalizerinfo`	显示在F-Queue中等待Finalizer线程执行finalize方法的对象，**仅linux/solaris平台有效**
- `-permstat`	以ClassLoader为统计口径输出永久代的内存状态信息，**仅linux/solaris平台有效**
- `-F`	当虚拟机进程对`-dump`选项没有任何响应时，强制执行生成dump文件，**仅linux/solaris平台有效**


#### 导出内存映像文件
- 手动导出
  - `jmap -dump:format=b,file=<filename.hprof> <pid>`
    - 例`jmap -dump:format=b,file=test.hprof 9564`
  - `jmap -dump:live,format=b,file=<filename.hprof> <pid>`
    - 例`jmap -dump:live,format=b,file=test2.hprof 9564`

?> `format=b` 以二进制hprof格式转储Java 堆

> 程序发生OOM时，一些瞬时信息都随着程序的终止而消失，而重现OOM往往比较困难或耗时久，所以可以使用JVM参数配置，在程序崩溃时自动导出当前堆快照   

- 自动导出(JVM参数)
  - `-XX:+HeapDumpOnOutOfMemoryError` 程序发生OOM时，导出应用程序当前堆快照
  - 若只使用上面参数默认位置为项目位置根目录，文件名`java_<pid>.hprof`
  - `-XX:+HeapDumpPath=<filename.hprof>` 指定快照保存位置和文件名默认文件名为`java_<pid>_<date>_<time>_heapDump.hprof`

> 通常在写Heap Dump文件前会触发一次FullGC，所以heap dump文件里保存的都是FullGC后留下的对象信息   
> **生成dump文件比较耗时**  



#### 显示堆内存相关信息
- `jmap -heap 进程id` 输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用信息等
- `jmap -histo 进程id`输出堆空间中对象的统计信息，包括类、实例数量和合计容量，
  - `-histo:live`只统计堆中的存活对象

> 由于jmap将访问堆中的所有对象，为了保证在此过程中不被应用线程干扰，jmap需要借助安全点机制，让所有线程停留在不改变堆中数据的状态。也就是说，由jmap导出的堆快照必定是安全点位置的。这可能导致基于该堆快照的分析结果存在偏差。

> 举个例子，假设在编译生成的机器码中，某些对象的生命周期在两个安全点之间，那么:live选项将无法探知到这些对象。    
> 另外，如果某个线程长时间无法跑到安全点，jmap将一直等下去。与前面讲的jstat则不同，垃圾回收器会主动将jstat所需要的摘要数据保存至固定位置之中，而jstat只需直接读取即可。

### jhat 
jhat`JVM Heap Analysis Tool`Sun JDK提供的**jhat命令与jmap命令搭配使用，用于分析jmap生成的heap dump文件**（堆转储快照）。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，用户可以在浏览器中查看分析结果（分析虚拟机转储快照信息）。

使用了jhat命令，就启动了一个http服务，端口是7000，即http://localhost:7000/，就可以在浏览器里分析。   
说明：**jhat命令在JDK9、JDK10中已经被删除，官方建议用VisualVM代替**。

基本适用语法：`jhat <option> <dumpfile>`例` jhat .\test.hprof`，启动后访问http://localhost:7000/   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jhat示例.png)

option参数   
- `-stack false｜true`	关闭｜打开对象分配调用栈跟踪
- `-refs false｜true`	关闭｜打开对象引用跟踪
- `-port port-number`	设置jhat HTTP Server的端口号，默认7000
- `-exclude exclude-file`	执行对象查询时需要排除的数据成员
- `-baseline exclude-file`	指定一个基准堆转储
- `-debug int`	设置debug级别
- `-version`	启动后显示版本信息就退出
- `-J <flag>`	传入启动参数，比如`-J -Xmx512m`

### jstack
jstack`JVM Stack Trace`：用于**生成虚拟机指定进程当前时刻的线程快照（虚拟机堆栈跟踪）**。线程快照就是当前虚拟机内指定进程的每一条线程正在执行的方法堆栈的集合。

生成线程快照的作用：可用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等问题。这些都是导致线程长时间停顿的常见原因。当线程出现停顿时，就可以**用jstack显示各个线程调用的堆栈情况**。

[官方文档](https://docs.oracle.com/en/java/javase/11/tools/jstack.html)

在thread dump中，要留意下面几种状态  
- 死锁，Deadlock（重点关注）
- 等待资源，Waiting on condition（重点关注）
- 等待获取监视器，Waiting on monitor entry（重点关注）
- 阻塞，Blocked（重点关注）
- 执行中，Runnable
- 暂停，Suspended
- 对象等待中，Object.wait() 或 TIMED＿WAITING
- 停止，Parked

> 例：`jps` 然后使用 `jstack pid` 查看

- `-F`	当正常输出的请求不被响应时，强制输出线程堆栈
- `-l`	除堆栈外，显示关于锁的附加信息
- `-m`	如果调用本地方法的话，可以显示C/C++的堆栈

### jcmd
在JDK 1.7以后，新增了一个命令行工具jcmd。它是一个**多功能的工具**，可以用来实现前面除了jstat之外所有命令的功能。   
比如：用它来导出堆、内存使用、查看Java进程、导出线程信息、执行GC、JVM运行时间等。    
[官方文档](https://docs.oracle.com/en/java/javase/11/tools/jcmd.html)

- `jcmd`拥有`jmap`的大部分功能，并且在Oracle的官方网站上也推荐使用`jcmd`命令代`jmap`命令
- `jcmd -l`：列出所有的JVM进程相当与`jps`
- `jcmd pid help`：针对指定的进程，列出支持的所有具体命令

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jcmd-help示例.png)

`jcmd 15744 GC.heap_dump /test.hprof` 测试生成dump文件 
`jcmd 15744 GC.heap_dump d:/test.hprof` 

- `Thread.print` 可以替换 jstack指令
- `GC.class_histogram` 可以替换 jmap中的-histo操作
- `GC.heap_dump` 可以替换 jmap中的-dump操作
- `GC.run` 可以查看GC的执行情况
- `VM.uptime` 可以查看程序的总执行时间，可以替换jstat指令中的-t操作
- `VM.system_properties` 可以替换 `jinfo -sysprops` pid
- `VM.flags` 可以获取JVM的配置参数信息

### jstatd
之前的指令只涉及到监控本机的Java应用程序，而在这些工具中，一些监控工具也支持对远程计算机的监控（如jps、jstat）。为了启用远程监控，则需要配合使用jstatd 工具。命令jstatd是一个RMI服务端程序，它的作用相当于代理服务器，建立本地计算机与远程监控工具的通信。jstatd服务器将本机的Java应用程序信息传递到远程计算机

[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)

## GUI工具
使用上一章命令行工具或组合能帮您获取目标Java应用性能相关的基础信息，但它们存在下列局限：  
- 无法获取方法级别的分析数据，如方法间的调用关系、各方法的调用次数和调用时间等（这对定位应用性能瓶颈至关重要）。
- 要求用户登录到目标 Java 应用所在的宿主机上，使用起来不是很方便。
- 分析数据通过终端输出，结果展示不够直观。

为此，JDK提供了一些内存泄漏的分析工具，如jconsole，jvisualvm等，用于辅助开发人员定位问题，但是这些工具很多时候并不足以满足快速定位的需求。所以这里我们介绍的工具相对多一些、丰富一些。

JDK自带的工具   
-  `jconsole`：JDK自带的可视化监控工具。查看Java应用程序的运行概况、监控堆信息、永久区（或元空间）使用情况、类加载情况等 
-  `Visual VM：Visual VM`是一个工具，它提供了一个可视界面，用于查看Java虚拟机上运行的基于Java技术的应用程序的详细信息。 
-  `JMC：Java Mission Control`，内置Java Flight Recorder。能够以极低的性能开销收集Java虚拟机的性能数据。 

第三方工具  
-  `MAT`：MAT（Memory Analyzer Tool）是基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗 
-  `JProfiler`：商业软件，需要付费。功能强大。
-  `Arthas`阿里开源诊断工具 



### JConsole
jconsole：从Java5开始，在JDK中自带的java监控和管理控制台。用于对JVM中内存、线程和类等的监控，是一个基于JMX（java management extensions）的GUI性能监控工具。

[官方文档](https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jconsole主界面.png)

可以在线程选项卡检测死锁 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jconsole死锁检测.png)


### J Visual VM
Visual VM是一个功能强大的多合一故障诊断和性能监控的可视化工具。它集成了多个JDK命令行工具，使用Visual VM可用于显示虚拟机进程及进程的配置和环境信息（jps，jinfo），监视应用程序的CPU、GC、堆、方法区及线程的信息（jstat、jstack）等，甚至代替JConsole。在JDK 6 Update 7以后，Visual VM便作为JDK的一部分发布（VisualVM 在JDK／bin目录下）即：它完全免费。

主要功能：
- 1.生成/读取堆内存/线程快照
- 2.查看JVM参数和系统属性
- 3.查看运行中的虚拟机进程
- 4.程序资源的实时监控
- 5.JMX代理连接、远程环境监控、CPU分析和内存分析

[官方文档](https://visualvm.github.io/index.html)


### Eclipse MAT

MAT（Memory Analyzer Tool）工具是一款功能强大的Java堆内存分析器。可以用于查找内存泄漏以及查看内存消耗情况。MAT是基于Eclipse开发的，不仅可以单独使用，还可以作为插件的形式嵌入在Eclipse中使用。是一款免费的性能分析工具，使用起来非常方便。

MAT可以分析heap dump文件。在进行内存分析时，只要获得了反映当前设备内存映像的hprof文件，通过MAT打开就可以直观地看到当前的内存信息。一般说来，这些内存信息包含：

- 所有的对象信息，包括对象实例、成员变量、存储于栈中的基本类型值和存储于堆中的其他对象的引用值。
- 所有的类信息，包括classloader、类名称、父类、静态变量等
- GCRoot到所有的这些对象的引用路径
- 线程信息，包括线程的调用栈及此线程的线程局部变量（TLS）

能够快速为开发人员生成内存泄漏报表，方便定位问题和分析问题。虽然MAT有如此强大的功能，但是内存分析也没有简单到一键完成的程度，很多内存问题还是需要我们从MAT展现给我们的信息当中通过经验和直觉来判断才能发现。

[官方文档](https://www.eclipse.org/mat/downloads.php)

?> 下载注意和JDK版本对应的版本，否则不兼容  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MAT总体示意图.png)

#### 分析堆dump文件

- histogram 展示了各个类的实例数目以及这些实例的Shallow heap(浅堆)或者Retained heap(深堆)的总和

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MAThistogram示意图.png)

- thread overview 查看系统中的Java线程，局部变量的信息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/thread-overview线程信息查看.png)

- 支配树，Dominator Tree 支配树体现了对象实例之间的支配关系

#### 溢出分析过程
1. 首先得到dump文件导入MAT进入，看到overview大概情况，找到最大占用

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Mat堆溢出分析1.png)

2. 点击最大占用，选择outgoing(出引用) 查看引用了那些内容  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Mat堆溢出分析2.png)

3. 分析出引用内容发现，有一个引用占用了绝大部分空间

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Mat堆溢出分析3.png)

4. 展开内部发现是byte数组内容过多

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Mat堆溢出分析4.png)

### JProfiler
在运行Java的时候有时候想测试运行时占用内存情况，这时候就需要使用测试工具查看了。在eclipse里面有 Eclipse Memory Analyzer tool（MAT）插件可以测试，而在IDEA中也有这么一个插件，就是JProfiler。JProfiler 是由 ej-technologies 公司开发的一款 Java 应用性能诊断工具。功能强大，但是**收费**。

特点：

- 使用方便、界面操作友好（简单且强大）
- 对被分析的应用影响小（提供模板）
- CPU，Thread，Memory分析功能尤其强大
- 支持对jdbc，noSql，jsp，servlet，socket等进行分析
- 支持多种模式（离线，在线）的分析
- 支持监控本地、远程的JVM
- 跨平台，拥有多种操作系统的安装版本

主要功能：

- 1-方法调用：对方法调用的分析可以帮助您了解应用程序正在做什么，并找到提高其性能的方法
- 2-内存分配：通过分析堆上对象、引用链和垃圾收集能帮您修复内存泄露问题，优化内存使用
- 3-线程和锁：JProfiler提供多种针对线程和锁的分析视图助您发现多线程问题
- 4-高级子系统：许多性能问题都发生在更高的语义级别上。例如，对于JDBC调用，您可能希望找出执行最慢的SQL语句。JProfiler支持对这些子系统进行集成分析

[官网](https://www.ej-technologies.com/products/jprofiler/overview.html)


### Arthas
上述工具都必须在服务端项目进程中配置相关的监控参数，然后工具通过远程连接到项目进程，获取相关的数据。这样就会带来一些不便，比如线上环境的网络是隔离的，本地的监控工具根本连不上线上环境。

那么有没有一款工具不需要远程连接，也**不需要配置监控参数，同时也提供了丰富的性能监控数据呢**？

阿里巴巴开源的性能分析神器Arthas应运而生。

Arthas是Alibaba开源的Java诊断工具，深受开发者喜爱。在线排查问题，无需重启；动态跟踪Java代码；实时监控JVM状态。Arthas 支持JDK 6＋，支持Linux／Mac／Windows，采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：


[官方地址](https://arthas.aliyun.com/doc/quick-start.html)

安装方式：如果速度较慢，可以尝试国内的码云Gitee下载。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Arthas启动实例.png)

启动成功后可以直接访问http://127.0.0.1:8563/  

命令列表详见 https://arthas.aliyun.com/doc/commands.html  

JVM命令  
- `dashboard`总览面板 
  - `dashboard -i 1000 -n 4` 没1000ms打印一次，共打印4次  
- `thread`当前JVM的线程堆栈信息 
  - 之后使用`thread ID` 查看指定id的线程信息
  - `thread b` 线程block信息
  - `thread -i 5000` 五秒内的线程统计 
  - `thread -n 5`cpu占用前五的线程
- `sysprop` 查看和修改JVM的系统属性
- `heapdump` 
  - `heapdump ./oom.hprof` 生成dump文件和jar包同目录
  - `heapdump --live ./oom.hprof` live对象的dump

ClassLoader   

- sc 查看JVM已加载的类信息
  - -d 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的Classloader等详细信息。如果一个类被多个Classloader所加载，则会出现多次
  - -E 开启正则表达式匹配，默认为通配符匹配
  - -f 输出当前类的成员变量信息（需要配合参数-d一起使用）
  - -X 指定输出静态变量时属性的遍历深度，默认为0，即直接使用toString输出
  - `sc -d -f java.lang.String`查看String内部的方法成员变量
- sm 查看已加载类的方法信息
  - -d 展示每个方法的详细信息
  - -E 开启正则表达式匹配,默认为通配符匹配
- jad 反编译指定已加载类的源码
- mc 内存编译器，内存编译.java文件为.class文件
- retransform 加载外部的.class文件, retransform到JVM里
- redefine 加载外部的.class文件，redefine到JVM里

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Arthasmc命令redefine命令.png)

- dump dump已加载类的byte code到特定目录
- classloader 查看classloader的继承树，urts，类加载信息，使用classloader去getResource
  - -t 查看classloader的继承树
  - -l 按类加载实例查看统计信息
  - -c 用classloader对应的hashcode来查看对应的 Jar urls


- monitor 方法执行监控，调用次数、执行时间、失败率
  - -c 统计周期，默认值为120秒
- watch 方法执行观测，能观察到的范围为：返回值、抛出异常、入参，通过编写groovy表达式进行对应变量的查看
	- -b 在方法调用之前观察(默认关闭)
	- -e 在方法异常之后观察(默认关闭)
	- -s 在方法返回之后观察(默认关闭)
	- -f 在方法结束之后(正常返回和异常返回)观察(默认开启)
	- -x 指定输岀结果的属性遍历深度,默认为0
- trace 方法内部调用路径,并输出方法路径上的每个节点上耗时
	- -n 执行次数限制
- stack 输出当前方法被调用的调用路径
- tt 方法执行数据的时空隧道,记录下指定方法每次调用的入参和返回信息,并能对这些不同的时间下调用进行观测

### Flame Graphs火焰图
在追求极致性能的场景下，了解你的程序运行过程中cpu在干什么很重要，火焰图就是一种非常直观的展示CPU在程序整个生命周期过程中时间分配的工具。火焰图对于现代的程序员不应该陌生，这个工具可以非常直观的显示出调用找中的CPU消耗瓶颈。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/火焰图示例.png)

火焰图，简单通过x轴横条宽度来度量时间指标，y轴代表线程栈的层次。



##  OQL语言查询
OQL语言使用类SQL语法, 可以在堆中进行对象的查找和筛选, 不区分大小写

### SELECT子句

```sql
SELECT * FROM java.util.Vector v

/* 使用OBJECTS 关键字, 可以将返回的结果集中的项以对象的形式显示 */
SELECT OBJECTS v.elementData FROM java.util.Vector v

/* 在SELECT子句中, 使用 AS RETAINED SET 关键字可以得到所有对象的保留集 */
SELECT AS RETAINED SET v.elementData FROM java.util.Vector v

/* 使用DISTINCT 关键字用于在结果集中去除重复对象 */
SELECT DISTINCT OBJECTS classof(v) FROM java.util.Vector v

```

### FROM子句

```sql
/*类名*/
SELECT * FROM java.util.Vector v
/*地址, 使用地址的好处是可以区分不同ClassLoader加载的同一种类型*/
SELECT * FROM 0xd44f3080
/*正则 查询java.lang包下的所有类的实例*/
SELECT * FROM "java\.lang\..*"


```

### WHERE子句

```sql
/* 查询长度大于10的数组 */
SELECT * FROM char[] s WHERE s.@length>10

/* 查询包含"java"字符串的所有Vector, 使用LIKE操作符 参数为正则表达式 */
SELECT * FROM java.util.Vector v WHERE toString(v) LIKE ".*java.*"

/* 查询所有value域不为null的字符串, 使用=号 */
SELECT * FROM java.lang.String  s where s.value!=null

/* 查询数组长度大于15 并且深堆 大于1000字节的所有Vector对象 */
SELECT * FROM java.util.Vector v WHERE v.elementData.@length >15 AND v.@retainedHeapSize>1000

```

### 内置对象与方法
- OQL可以访问堆内对象的属性, 也可以 访问堆内代理对象的属性
- 访问堆内对象属性的格式: `<alias>.<field>.<field>.<field>`, 其中alias为对象名称

```sql
/* 访问 java.io.File 对象的Path属性 并进一步 访问其value */
SELECT toString(f.path.value) FROM java.io.File f

/* 查询String对象的内容, objectId和objectAddress */
SELECT s.toString(),s.@objectId,s.@objectAddress FROM java.lang.String s

/* 显示所有Vector对象及其子类型 */
SELECT * FROM INSTANCEOF java.util.Vector v

```

## 浅堆和深堆

### 浅堆（Shollow Heap）
浅堆（Shallow Heap）是指一个对象所消耗的内存。在 32 位系统中，一个对象引用会占据 4 个字节，一个 int 类型会占据 4 个字节，long 型变量会占据 8 个字节，每个对象头需要占用 8 个字节。根据堆快照格式不同，对象的大小可能会向 8 字节进行对齐

以 String 为例：2 个 int 值共占 8 字节， 对象引用占用 4 字节，对象头 8 字节，合计 20 字节，向 8 字节对齐，故占 24 字节（jdk7中）

这 24 字节为 String 对象的浅堆大小。它与 String 的 value 实际取值无关，无论字符串长度如何，浅堆大小始终是 24 字节

### 保留集（Retained Set）
对象 A 的保留集指当对象 A 被垃圾回收后，可以被释放的所有的对象集合（包括对象A本身），即对象 A 的保留集可以被认为是只能通过对象 A 被直接或间接访问到的所有对象的集合。通俗地说，就是指仅被对象 A 所持有的对象的集合

### 深堆（Retained Heap）
**深堆是指对象的保留集中所有的对象的浅堆大小之和**

注意：浅堆指对象本身占用的内存，不包括其内部引用对象的大小。一个对象的深堆指只能通过该对象访问到的（直接或间接）所有对象的浅堆之和，即对象被回收后，可以释放的真实空间

### 对象实际大小
另外一个常用的概念是对象的实际大小。这里，对象的实际大小定义为一个对象所能触及的所有对象的浅堆大小之和，也就是通常意义上我们说的对象大小。与深堆相比，似乎这个在日常开发中更为直观和被人接受，但实际上，这个概念和垃圾回收无关

下图显示了一个简单的对象引用关系图，对象 A 引用了 C 和 D，对象 B 引用了 C 和 E。那么对象 A 的浅堆大小只是 A 本身，不含 C 和 D，而 A 的实际大小为 A、C、D 三者之和。而 A 的深堆大小为 A 与 D 之和，由于对象 C 还可以通过对象 B 访问到，因此不在对象 A 的深堆范围内

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/浅堆深堆示例.png)



# 运行时参数

## 标准参数选项

```shell
> java -help
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32          使用 32 位数据模型 (如果可用)
    -d64          使用 64 位数据模型 (如果可用)
    -server       选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 ; 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
    -version:<值>
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  需要指定的版本才能运行
    -showversion  输出产品版本并继续
    -jre-restrict-search | -no-jre-restrict-search
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  在版本搜索中包括/排除用户专用 JRE
    -? -help      输出此帮助消息
    -X            输出非标准选项的帮助
    -ea[:<packagename>...|:<classname>]
    -enableassertions[:<packagename>...|:<classname>]
                  按指定的粒度启用断言
    -da[:<packagename>...|:<classname>]
    -disableassertions[:<packagename>...|:<classname>]
                  禁用具有指定粒度的断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<libname>[=<选项>]
                  加载本机代理库 <libname>, 例如 -agentlib:hprof
                  另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
    -agentpath:<pathname>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jarpath>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<imagepath>
                  使用指定的图像显示启动屏幕
有关详细信息, 请参阅 http://www.oracle.com/technetwork/java/javase/documentation/index.html。
```

## Server模式和Client模式

Hotspot JVM有两种模式，分别是server和client，分别通过-server和-client模式设置

- 32位系统上，默认使用Client类型的JVM`C1`。要想使用Server模式，机器配置至少有2个以上的CPU和2G以上的物理内存。client模式适用于对内存要求较小的桌面应用程序，默认使用Serial串行垃圾收集器
- 64位系统上，只支持server模式的JVM`C2`，适用于需要大内存的应用程序，默认使用并行垃圾收集器

[官网介绍地址](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/server-class.html)

通过`java -version`命令：可以看到`Server VM`字样，代表当前系统使用是Server模式

## -X参数选项

```shell
> java -X
    -Xmixed           混合模式执行 (默认)
    -Xint             仅解释模式执行,禁用JIT编译器
    -Xcomp            强制使用编译模式
    -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                      设置搜索路径以引导类和资源
    -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                      附加在引导类路径末尾
    -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                      置于引导类路径之前
    -Xdiag            显示附加诊断消息
    -Xnoclassgc       禁用类垃圾收集
    -Xincgc           启用增量垃圾收集
    -Xloggc:<file>    将 GC 状态记录在文件中 (带时间戳)
    -Xbatch           禁用后台编译

    -Xms<size>        设置初始 Java 堆大小
    -Xmx<size>        设置最大 Java 堆大小
    -Xss<size>        设置 Java 线程堆栈大小
    
    -Xprof            输出 cpu 配置文件数据
    -Xfuture          启用最严格的检查, 预期将来的默认值
    -Xrs              减少 Java/VM 对操作系统信号的使用 (请参阅文档)
    -Xcheck:jni       对 JNI 函数执行其他检查
    -Xshare:off       不尝试使用共享类数据
    -Xshare:auto      在可能的情况下使用共享类数据 (默认)
    -Xshare:on        要求使用共享类数据, 否则将失败。
    -XshowSettings    显示所有设置并继续
    -XshowSettings:all
                      显示所有设置并继续
    -XshowSettings:vm 显示所有与 vm 相关的设置并继续
    -XshowSettings:properties
                      显示所有属性设置并继续
    -XshowSettings:locale
                      显示所有与区域设置相关的设置并继续

-X 选项是非标准选项, 如有更改, 恕不另行通知。
```

## -XX参数选项

```shell

-XX:+<option>  启用option属性
-XX:-<option>  禁用option属性

-XX:+UseG1GC 使用G1垃圾回收
-XX:-UseG1GC 禁用G1垃圾回收

-XX:<option>=<number>  设置option数值，可以带单位如k/K/m/M/g/G
-XX:<option>=<string>  设置option字符值

-XX:NewSize=1024m 新生代初始大小为1024m
-XX:MaxGCPauseMillis=500  GC停顿时间500ms
-XX:GCTimeRatio=19 设置吞吐量
-XX:NewRatio=2 新生代与老年代的比例

```

?> 可以使用`-XX:+PrintFlagsFinal`参数在程序结束时，打印所有参数值   
或者直接通过`java -XX:+PrintFlagsFinal -version`查看，这其中有**允许修改的参数**可以通过`java -XX:+PrintFlagsFinal -version | grep manageable`查看    
详见 **jinfo命令**


## 添加JVM参数选项

```shell
# 运行jar包
java -Xms100m -Xmx100m -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -jar demo.jar

# 运行war包 tomcat中
# linux下catalina.sh添加
JAVA_OPTS="-Xms512M -Xmx1024M"
# windows下catalina.bat添加
set "JAVA_OPTS=-Xms512M -Xmx1024M"
```

## 常用参数

### 打印设置的XX选项及值

```shell
-XX:+PrintCommandLineFlags 程序运行时JVM默认设置或用户手动设置的XX选项
-XX:+PrintFlagsInitial 打印所有XX选项的默认值
-XX:+PrintFlagsFinal 打印所有XX选项的实际值
-XX:+PrintVMOptions 打印JVM的参数
```

### 堆、栈、方法区等内存大小设置

```shell
# 栈
-Xss128k <==> -XX:ThreadStackSize=128k 设置线程栈的大小为128K

# 堆
-Xms2048m <==> -XX:InitialHeapSize=2048m 设置JVM初始堆内存为2048M
-Xmx2048m <==> -XX:MaxHeapSize=2048m 设置JVM最大堆内存为2048M
-Xmn2g <==> -XX:NewSize=2g -XX:MaxNewSize=2g 设置年轻代大小为2G
-XX:SurvivorRatio=8 设置Eden区与Survivor区的比值，默认为8
-XX:NewRatio=2 设置老年代与年轻代的比例，默认为2
-XX:+UseAdaptiveSizePolicy 设置大小比例自适应，默认开启，会导致实际分配eden 和s0,s1区比例为 6:1:1 
-XX:-UseAdaptiveSizePolicy -XX:SurvivorRatio=8  这两个参数同时使用才能使分配eden 和s0,s1区比例为 8:1:1 
-XX:PretenureSizeThreadshold=1024 设置让大于此阈值的对象直接分配在老年代，只对Serial、ParNew收集器有效
-XX:MaxTenuringThreshold=15 设置新生代晋升老年代的年龄限制，默认为15
-XX:TargetSurvivorRatio 设置MinorGC结束后Survivor区占用空间的期望比例

# 方法区
-XX:MetaspaceSize / -XX:PermSize=256m 设置元空间/永久代初始值为256M
-XX:MaxMetaspaceSize / -XX:MaxPermSize=256m 设置元空间/永久代最大值为256M
-XX:+UseCompressedOops 使用压缩对象
-XX:+UseCompressedClassPointers 使用压缩类指针
-XX:CompressedClassSpaceSize 设置Klass Metaspace的大小，默认1G

# 直接内存
-XX:MaxDirectMemorySize 指定DirectMemory容量，默认等于Java堆最大值
```

### OutOfMemory相关的选项

```shell
-XX:+HeapDumpOnOutMemoryError 内存出现OOM时生成Heap转储文件，针对程序突然去世的场景，和下面的二选一
-XX:+HeapDumpBeforeFullGC 出现FullGC时生成Heap转储文件，针对频繁fullgc 的场景，二选一

-XX:OnOutOfMemoryError=<path> 指定可行性程序或脚本的路径，当发生OOM时执行脚本
```

### 垃圾收集器相关选项
首先需了解垃圾收集器之间的搭配使用关系

- 红色虚线表示在jdk8时被Deprecate，jdk9时被删除
- 绿色虚线表示在jdk14时被Deprecate
- 绿色虚框表示在jdk9时被Deprecate，jdk14时被删除

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/垃圾收集器的组合关系.png ':size=60%')


```shell
# Serial回收器
-XX:+UseSerialGC  年轻代使用Serial GC， 老年代使用Serial Old GC
# ParNew回收器
-XX:+UseParNewGC  年轻代使用ParNew GC
-XX:ParallelGCThreads  设置年轻代并行收集器的线程数。
	一般地，最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。

# ParNewGC
-XX:+UseParNewGC 手动指定使用ParNew收集器执行内存回收任务，只影响新生代
-XX:ParallelGCThreads=N 限制线程数量，默认开启和CPU核心相同的线程数


# Parallel回收器
-XX:+UseParallelGC  年轻代使用 Parallel Scavenge GC，互相激活
-XX:+UseParallelOldGC  老年代使用 Parallel Old GC，互相激活

-XX:ParallelGCThreads=N 

-XX:MaxGCPauseMillis  设置垃圾收集器最大停顿时间（即STW的时间），单位是毫秒。
	为了尽可能地把停顿时间控制在MaxGCPauseMills以内，收集器在工作时会调整Java堆大小或者其他一些参数。
	对于用户来讲，停顿时间越短体验越好；但是服务器端注重高并发，整体的吞吐量。
	所以服务器端适合Parallel，进行控制。该参数使用需谨慎。
-XX:GCTimeRatio  垃圾收集时间占总时间的比例（1 / (N＋1)），用于衡量吞吐量的大小
	取值范围（0,100），默认值99，也就是垃圾回收时间不超过1％。
	与前一个-XX：MaxGCPauseMillis参数有一定矛盾性。暂停时间越长，Radio参数就容易超过设定的比例。

-XX:+UseAdaptiveSizePolicy  设置Parallel Scavenge收集器具有自适应调节策略。
	在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年代的对象年龄等参数会被自动调整，以达到在堆大小、吞吐量和停顿时间之间的平衡点。
	在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMills），让虚拟机自己完成调优工作。

# G1回收器
-XX:+UseG1GC 手动指定使用G1收集器执行内存回收任务。
-XX:G1HeapRegionSize 设置每个Region的大小。
	值是2的幂，范围是1MB到32MB之间，目标是根据最小的Java堆大小划分出约2048个区域。默认是堆内存的1/2000。
-XX:MaxGCPauseMillis  设置期望达到的最大GC停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms
-XX:ParallelGCThread  设置STW时GC线程数的值。最多设置为8
-XX:ConcGCThreads  设置并发标记的线程数。将n设置为并行垃圾回收线程数（ParallelGCThreads）的1/4左右。
-XX:InitiatingHeapOccupancyPercent 设置触发并发GC周期的Java堆占用率阈值。超过此值，就触发GC。默认值是45。
-XX:G1NewSizePercent  新生代占用整个堆内存的最小百分比（默认5％）
-XX:G1MaxNewSizePercent  新生代占用整个堆内存的最大百分比（默认60％）
-XX:G1ReservePercent=10  保留内存区域，防止 to space（Survivor中的to区）溢出


# CMS回收器
-XX:+UseConcMarkSweepGC  年轻代使用CMS GC。
	开启该参数后会自动将-XX：＋UseParNewGC打开。即：ParNew（Young区）+ CMS（Old区）+ Serial Old的组合
-XX:CMSInitiatingOccupanyFraction  设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收。JDK5及以前版本的默认值为68，DK6及以上版本默认值为92％。
	如果内存增长缓慢，则可以设置一个稍大的值，大的阈值可以有效降低CMS的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。
	反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。
	因此通过该选项便可以有效降低Fu1l GC的执行次数。
-XX:+UseCMSInitiatingOccupancyOnly  是否动态可调，使CMS一直按CMSInitiatingOccupancyFraction设定的值启动
-XX:+UseCMSCompactAtFullCollection  用于指定在执行完Full GC后对内存空间进行压缩整理
	以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。

-XX:CMSFullGCsBeforeCompaction  设置在执行多少次Full GC后对内存空间进行压缩整理。
-XX:ParallelCMSThreads  设置CMS的线程数量。
	CMS 默认启动的线程数是(ParallelGCThreads＋3)/4，ParallelGCThreads 是年轻代并行收集器的线程数。
	当CPU 资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕。
-XX:ConcGCThreads  设置并发垃圾收集的线程数，默认该值是基于ParallelGCThreads计算出来的
-XX:+CMSScavengeBeforeRemark  强制hotspot在cms remark阶段之前做一次minor gc，用于提高remark阶段的速度
-XX:+CMSClassUnloadingEnable  如果有的话，启用回收Perm 区（JDK8之前）
-XX:+CMSParallelInitialEnabled  用于开启CMS initial-mark阶段采用多线程的方式进行标记
	用于提高标记速度，在Java8开始已经默认开启
-XX:+CMSParallelRemarkEnabled  用户开启CMS remark阶段采用多线程的方式进行重新标记，默认开启
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
	这两个参数用户指定hotspot虚拟在执行System.gc()时使用CMS周期
-XX:+CMSPrecleaningEnabled  指定CMS是否需要进行Pre cleaning阶段

```

### GC日志相关选项

```shell
-XX:+PrintGC <==> -verbose:gc  打印简要日志信息
-XX:+PrintGCDetails            打印详细日志信息
-XX:+PrintGCTimeStamps  打印程序启动到GC发生的时间，搭配-XX:+PrintGCDetails使用才能生效
-XX:+PrintGCDateStamps  打印GC发生时的时间戳，搭配-XX:+PrintGCDetails使用
-XX:+PrintHeapAtGC  打印GC前后的堆信息
-Xloggc:<file> 输出GC导指定路径下的文件中  

-XX:+TraceClassLoading  监控类的加载
-XX:+PrintGCApplicationStoppedTime  打印GC时线程的停顿时间
-XX:+PrintGCApplicationConcurrentTime  打印垃圾收集之前应用未中断的执行时间
-XX:+PrintReferenceGC 打印回收了多少种不同引用类型的引用
-XX:+PrintTenuringDistribution  打印JVM在每次MinorGC后当前使用的Survivor中对象的年龄分布
-XX:+UseGCLogFileRotation 启用GC日志文件的自动转储
-XX:NumberOfGCLogFiles=1  设置GC日志文件的循环数目
-XX:GCLogFileSize=1M  设置GC日志文件的大小  


-XX:+TraceClassLoading  监控类的加载
-XX:+PrintGCApplicationStoppedTime  打印GC时线程的停顿时间
-XX:+PrintGCApplicationConcurrentTime  打印垃圾收集之前应用未中断的执行时间
-XX:+PrintReferenceGC 打印回收了多少种不同引用类型的引用
-XX:+PrintTenuringDistribution  打印JVM在每次MinorGC后当前使用的Survivor中对象的年龄分布
-XX:+UseGCLogFileRotation 启用GC日志文件的自动转储
-XX:NumberOfGCLogFiles=1  设置GC日志文件的循环数目
-XX:GCLogFileSize=1M  设置GC日志文件的大小


```

### 其他参数

```shell
-XX:+DisableExplicitGC  禁用hotspot执行System.gc()，默认禁用
-XX:ReservedCodeCacheSize=<n>[g|m|k]、-XX:InitialCodeCacheSize=<n>[g|m|k]  指定代码缓存的大小
-XX:+UseCodeCacheFlushing  放弃一些被编译的代码，避免代码缓存被占满时JVM切换到interpreted-only的情况
-XX:+DoEscapeAnalysis  开启逃逸分析
-XX:+UseBiasedLocking  开启偏向锁
-XX:+UseLargePages  开启使用大页面
-XX:+PrintTLAB  打印TLAB的使用情况
-XX:TLABSize  设置TLAB大小
```

### 通过Java代码获取JVM参数

> Java提供了`java.lang.management`包用于监视和管理Java虚拟机和Java运行时中的其他组件，它允许本地或远程监控和管理运行的Java虚拟机。   
> 其中`ManagementFactory`类较为常用，另外`Runtime`类可获取内存、CPU核数等相关的数据。通过使用这些api，可以监控应用服务器的堆内存使用情况，设置一些阈值进行报警等处理。

```java
public class MemoryMonitor {
    public static void main(String[] args) {
        MemoryMXBean memorymbean = ManagementFactory.getMemoryMXBean();
        MemoryUsage usage = memorymbean.getHeapMemoryUsage();
        System.out.println("INIT HEAP: " + usage.getInit() / 1024 / 1024 + "m");
        System.out.println("MAX HEAP: " + usage.getMax() / 1024 / 1024 + "m");
        System.out.println("USE HEAP: " + usage.getUsed() / 1024 / 1024 + "m");
        System.out.println("\nFull Information:");
        System.out.println("Heap Memory Usage: " + memorymbean.getHeapMemoryUsage());
        System.out.println("Non-Heap Memory Usage: " + memorymbean.getNonHeapMemoryUsage());

        System.out.println("=======================通过java来获取相关系统状态============================ ");
        System.out.println("当前堆内存大小totalMemory " + (int) Runtime.getRuntime().totalMemory() / 1024 / 1024 + "m");// 当前堆内存大小
        System.out.println("空闲堆内存大小freeMemory " + (int) Runtime.getRuntime().freeMemory() / 1024 / 1024 + "m");// 空闲堆内存大小
        System.out.println("最大可用总堆内存maxMemory " + Runtime.getRuntime().maxMemory() / 1024 / 1024 + "m");// 最大可用总堆内存大小

    }
}
```

# GC日志分析
之前的日志分析见[JVM性能监控](/Java/JVM性能监控.md)中`八、GC日志分析`  

## GC分类
针对HotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集`Partial GC`，一种是整堆收集`Full GC`

- 部分收集`Partial GC`：不是完整收集整个Java堆的垃圾收集。其中又分为： 
  -  新生代收集`Minor GC / Young GC`：只是新生代`Eden / S0, S1`的垃圾收集
  - 老年代收集`Major GC / Old GC`：只是老年代的垃圾收集。目前，只有CMS GC会有单独收集老年代的行为。注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。
 
- 混合收集`Mixed GC`：收集整个新生代以及部分老年代的垃圾收集。目前，只有`G1 GC`会有这种行为 
- 整堆收集`Full GC`：收集整个java堆和方法区的垃圾收集。 

## GC日志结构剖析

### 透过日志看垃圾收集器

-  Serial收集器：新生代显示 "[DefNew"，即 Default New Generation 
-  ParNew收集器：新生代显示 "[ParNew"，即 Parallel New Generation 
-  Parallel Scavenge收集器：新生代显示"[PSYoungGen"，JDK1.7使用的即PSYoungGen 
-  Parallel Old收集器：老年代显示"[ParoldGen" 
-  G1收集器：显示”garbage-first heap“ 

### 透过日志看GC原因
- `Allocation Failure`：表明本次引起GC的原因是因为新生代中没有足够的区域存放需要分配的数据
- `Metadata GCThreshold`：Metaspace区不够用了
- `FErgonomics`：JVM自适应调整导致的GC
- `System`：调用了System.gc()方法


### 透过日志看GC前后情况
可以发现GC日志格式的规律一般都是：GC前内存占用-＞GC后内存占用（该区域内存总大小）

```
[PSYoungGen: 5986K->696K (8704K) ] 5986K->704K (9216K)
```

-  中括号内：GC回收前年轻代堆大小，回收后大小，（年轻代堆总大小） 
-  括号外：GC回收前年轻代和老年代大小，回收后大小，（年轻代和老年代总大小） 

?> Minor GC堆内存总容量 = 9/10 年轻代 + 老年代。原因是Survivor区只计算from部分，而JVM默认年轻代中Eden区和Survivor区的比例关系，Eden:S0:S1=8:1:1

### 透过日志看GC时间

```
[GC (Allocation Failure) [PSYoungGen:136162K->5113K(136192K)] 141425K->17632K(222208K),0.0248249 secs] [Times:user=0.05 sys=0.00,real=0.03 secs]
```

GC日志中有三个时间(Times)：`user`，`sys`和`real`

-  `user`：进程执行用户态代码（核心之外）所使用的时间。这是执行此进程所使用的实际CPU 时间，其他进程和此进程阻塞的时间并不包括在内。在垃圾收集的情况下，表示GC线程执行所使用的 CPU 总时间。
-  `sys`：进程在内核态消耗的 CPU 时间，即在内核执行系统调用或等待系统事件所使用的CPU 时间
-  `real`：程序从开始到结束所用的时钟时间。这个时间包括其他进程使用的时间片和进程阻塞的时间（比如等待 I/O 完成）。对于并行gc，这个数字应该接近（用户时间＋系统时间）除以垃圾收集器使用的线程数。

由于多核的原因，一般的GC事件中，real time是小于sys time＋user time的，因为一般是多个线程并发的去做GC，所以real time是要小于sys＋user time的。如果real＞sys＋user的话，则你的应用可能存在下列问题：IO负载非常重或CPU不够用


##  GC日志分析工具

**GCEasy**

GCEasy是一款在线的GC日志分析器，可以通过GC日志分析进行内存泄露检测、GC暂停原因分析、JVM配置建议优化等功能，大多数功能是免费的。  
官网地址：https://gceasy.io/

**GCViewer**

GCViewer是一款离线的GC日志分析器，用于可视化Java VM选项 -verbose:gc 和 .NET生成的数据 -Xloggc:<file>。还可以计算与垃圾回收相关的性能指标（吞吐量、累积的暂停、最长的暂停等）。当通过更改世代大小或设置初始堆大小来调整特定应用程序的垃圾回收时，此功能非常有用。  

源码下载：https://github.com/chewiebug/GCViewer    
运行版本下载：https://github.com/chewiebug/GCViewer/wiki/Changelog    

**GChisto**

- 官网上没有下载的地方，需要自己从SVN上拉下来编译
- 不过这个工具似乎没怎么维护了，存在不少bug

**HPjmeter**

- 工具很强大，但是只能打开由以下参数生成的GC log，-verbose:gc  -Xloggc:gc.log。添加其他参数生成的gc.log无法打开
- HPjmeter集成了以前的HPjtune功能，可以分析在HP机器上产生的垃圾回收日志文件
























# 参考资料
> - [尚硅谷JVM](https://www.bilibili.com/video/BV1PJ411n7xZ)
> - [Java 虚拟机底层原理知识总结](https://doocs.gitee.io/jvm/)
> - [《深入理解JVM虚拟机第三版》]()
> - [《深入理解JVM虚拟机第三版》的勘误仓库](https://github.com/fenixsoft/jvm_book)
> - [OQL语言参考](https://blog.csdn.net/Ty_0026/article/details/113877953)