---
layout: post
title:  jvm monitor tool learn
date: 2020-02-20
tag: java
---
<br>

## jvm monitor tool learn  



### 监控集成工具  
* VisualVm
* jConsole

### jdk 监控命令  

* jps 查看运行的java进程  

````
jps -m -l -v
-m 输出传入main方法的参数
-l 输出main类或jar的全限名
-v 输出传入的jvm参数

/opt/running # jps -lmv
1 bootshiro.jar --spring.profiles.active=prod
97 jdk.jcmd/sun.tools.jps.Jps -lmv -Dapplication.home=/opt/openjdk-12 -Xms8m -Djdk.module.main=jdk.jcmd

````

* jstack 查看java进程的线程dump  

````
jstack -l -e <pid>
-l      长列表，打印锁附加信息
-e      扩展列表，打印线程附件信息
<pid>   java进程ID,通过jps查看

/opt/running # jstack -l -e 1
2020-02-20 13:17:23
Full thread dump OpenJDK 64-Bit Server VM (12-ea+29 mixed mode, sharing):


````

* jmap dump转储堆栈信息  

````
jmap -dump:live,format=b,file=heap.hprof <pid>
-dump:live       转储活动的对象，如果不指定(去掉live)就是所有对象
format=b         二进制格式
file=heap.bin    转储生成文件heap.bin
<pid>            java进程ID

/opt/running # jmap -dump:,format=b,file=heap.hprof 1
Heap dump file created

转储生成的dump文件可以导入VisualVm，jConsole或者jhat命令查看

/opt/running # jhat heap.hprof
Reading from heap.bin...
Dump file created Thu Feb 20 21:29:23 CST 2020
Snapshot read, resolving...
Resolving 660870 objects...
Chasing references, expect 132 dots...
Eliminating duplicate references...
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
通过localhost:7000查看

````

* jstat JVM统计监测工具  

````
jstat --help|-options
jstat -<option> [-h<lines>] <vmid> [<interval> [<count>]]
-<option>     选项，通过jstat -options获取
[-h<lines>]   可选项，表示多少行之后再次打印头部信息
<vmid>        监控的java进程ID
[<interval>]  可选项，打印间隔，表示每隔多少ms打印一次信息
[<count>]     可选项，打印次数，表示打印几次就停止

下面表示: 分析进程ID为1的进程的GC情况，2行之后就再次打印头部，每隔2秒打印一次，总打印4次
/opt/running # jstat -gc -h2 1 2000 4
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT
2816.0 2816.0  0.0   597.8  22528.0  18711.1   56028.0    33616.4   80680.0 78501.6 9336.0 8607.4   1553    5.452   4      0.757   -          -    6.209
2816.0 2816.0  0.0   597.8  22528.0  18711.1   56028.0    33616.4   80680.0 78501.6 9336.0 8607.4   1553    5.452   4      0.757   -          -    6.209
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT
2816.0 2816.0  0.0   597.8  22528.0  18711.1   56028.0    33616.4   80680.0 78501.6 9336.0 8607.4   1553    5.452   4      0.757   -          -    6.209
2816.0 2816.0  0.0   597.8  22528.0  18711.1   56028.0    33616.4   80680.0 78501.6 9336.0 8607.4   1553    5.452   4      0.757   -          -    6.209


查看option选项有哪些可选参数
/opt/running # jstat -options
-class                显示加载的class数量，所占空间，耗时
-compiler             显示编译的数量,失败，失败的内容等信息
-gc                   (频繁)显示GC信息，年轻代，老年代，gc次数等信息
-gccapacity           显示年轻代老年代永久代的内存占用大小等信息
-gccause              (频繁)显示最近一次GC信息和GC原因
-gcmetacapacity       显示元空间容量信息
-gcnew                显示年轻代信息
-gcnewcapacity        显示年轻代信息及占用量
-gcold                显示老年代信息
-gcoldcapacity        显示老年代信息及占用量
-gcutil               显示GC的统计信息

````

````
/opt/running # jstat -gc -h2 1 2000 4
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT    CGC    CGCT     GCT
2816.0 2816.0  0.0   597.8  22528.0  18711.1   56028.0    33616.4   80680.0 78501.6 9336.0 8607.4   1553    5.452   4      0.757   -          -    6.209
````

jstat -gc | 参数解释
----------|---------
S0C       | 年轻代8:1:1的第一个幸存区
S1C       | 年轻代8:1:1的第二个幸存区
S0U       | 第一个幸存区使用大小
S1U       | 第二个幸存区使用大小
EC        | 年轻代8:1:1中的伊甸园大小
EU        | 伊甸园使用大小
OC        | 老年代大小
OU        | 老年代使用大小
MC        | 方法区(元空间)大小
MC        | 方法区(元空间)使用大小
CCSC      | 压缩类空间？？大小  [压缩类](https://www.zhihu.com/question/268392125)
CCSU      | 压缩类空间使用大小
YGC       | 发生的young gc次数
YGCT      | young gc所耗时间
FGC       | 发生的full gc次数
FGCT      | full gc所耗时间
CGC       | 并发垃圾搜集次数(G1垃圾搜集器)
CGCT      | 并发垃圾搜集耗时
GCT       | 总耗gc时间


### 分析java占用cpu或内存高的线程  

* `top` 查看高内存或高cpu的进程pid(大写P排序cpu，大写M排序内存)  
* `jps` 查看要看的java进程pid  

* `top -Hp <pid>` 列出该pid进程下的所有线程信息(列表中PID为线程十进制id)  
* 通过(大写P排序cpu，大写M排序内存)查出占用高的线程id  
* `printf "%x\n" <threadId>`转16进制  
* `jstack <pid> | grep <threadId>` 查看该线程堆栈  

### jvm奔溃hs_err_pid.log分析  

* 默认当前在工作目录，通过 ```-XX:ErrorFile=/opt/log/hs_err_pid<pid>.log``` 设定生成路径  
* 相关文章 [JVM致命错误日志(hs_err_pid.log)分析](https://blog.csdn.net/github_32521685/article/details/50355661)

### arthas生产环境定位  

* watch观测方法调用前后数据  

````
watch com.usthe.sureness.WatchClass demoMethod "{params,target,returnObj}" -x 3 -b -s  
params      - 方法参数信息
target      - 方法所在对象的参数信息(对象变量,类变量等..)
returnObj   - 方法返回值信息  
-x          - 表示对对象值的遍历深度（list对象深度1只能看到对象内容，深度2能看到参数数量等信息，深度3能看到list包含的对象的具体值信息）-x 3表示遍历深度3  
-b          - 表示观测方法调用前  
-s          - 表示观测方法调用后  
````

### 端口情况分析  

````
netstat -an | awk '{print $6}' | sort | uniq -c | sort -rn         - 查看端口各个状态的数量
````

### jvm监控相关好文  

* [JVM监控命令详解](https://www.cnblogs.com/rainy-shurun/p/5732341.html)
* [如何使用jstack分析线程状态](https://www.cnblogs.com/wuchanming/p/7766994.html)
* [Java语言定义的线程状态分析](https://www.cnblogs.com/trust-freedom/p/6606594.html)




<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
