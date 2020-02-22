---
layout: post
title:  Java thread learn
date: 2020-02-22
tag: java
---
<br>

### java线程状态学习

<br>
<br>


![线程状态图](/images/posts/java_thread/thread_state.svg)

#### 线程状态  

* ```NEW```                            新建，未start的线程
  1. 继承Thread
  2. 实现Runnable
  3. 实现Callable注入FutureTask

* ```RUNNABLE```                可运行，cpu调度运行(运行中，就绪)
  1. start()

* ```WAITING```                  无限期等待，主动进入
  1. Object.wait()
  2. Thread.join() 其会用Object.wait()实现
  3. LockSupport.park()
  4. 其他使用上面三个来同步控制的(ReentrantLock)

* ```TIME_WAITING```       有限等待，主动进入
  1. Object.wait(long_time)
  2. Thread.sleep(long_time)
  3. Thread.join(long_time)  其会用Object.wait()实现
  4. LockSupport.parkNanos(long_time)
  5. LocksUPPORT.parkUntil(long_deadtime)   deadtime - 终止时间点
  6. 其他使用上面四个来同步控制的
  
* ```BLOCKED```                  阻塞，被动进入
  1. enter synchronized
  2. reenter synchronzied (获取监视器对象锁后Object.wait()后被notify唤醒重新进入synchronized，再次去抢占获取监视器对象锁)    

* ```TERMINATED```           终止
  1. 程序跑完
  2. 抛出异常


#### 堆栈线程状态  

java线程状态                              | 堆栈线程状态
---                                       | ---
NEW                                       | 未运行无此状态
RUNNABLE                                  | RUNNABLE
WAITING (wait(),join())                   | WAITING (on object monitor)
WAITING (park())                          | WAITING (parking)
TIMED_WAITING (sleep(t))                  | TIMED_WAITING (sleeping)
TIMED_WAITING (wait(t),join(t))           | TIMED_WAITING (on object monitor)
TIMED_WAITING (parkNanos(t),parkUntil(t)  | TIMED_WAITING (parking)
BLOCKED (synchronzied)                    | BLOCKED (on object monitor)


<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
