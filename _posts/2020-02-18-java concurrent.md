---
layout: post
title:  Java Concurrent Learn
date: 2020-02-18
tag: java
---

<br>
<br>

## Java Concurrent Learn       

对java并发的一些学习,之前总结在幕布上,为了加深映像,这里重新学习下。  

<br>

### 并发级别  

* 阻塞 - 对临界区的代码串行化执行
  1. synchronized
  2. ReentrantLock
* 无饥饿 - 在优先级并发中,可能低优先级的永远不能执行的情况,需要避免就叫无饥饿
  1. 队列排队请求资源
  2. 公平锁
* 无障碍 - 读写锁控制力度不同,这里主要是乐观读
  1. 乐观锁-一致性标记,状态版本控制
* 无锁 - 比较交换,不是真正的不加锁,而是用java本地Unsaft 的cas方法
  1. CAS - compare and swap
* 无等待 - 读写不等待,在写的时候,重新复制一个对象对其操作,操作完之后在合适的时机赋值回去,这种只适合极少量写,大量读的情况
  1. RCU - read copy update  

### 并发原则 - 三大原则  

* 原子性 - atomic*原子操作类
* 可见性 - volatile关键字保证
* 有序性 - happen-before规则

### 线程创建  

* 实现Runnable接口- 其实所有创建方式都是需要实现Runnable接口  
````
    public class RunnableTest implements Runnable {
        @Override
        public void run() {}
    }
````

* 继续Thread  
````
    public class ThreadTest extends Thread {
        @Override
        public void run() {}
    }
````

* 用Callable接口实现类构造FutureTask  
````
    public class CallableTest implements Callable {
        @Override
        public Object call() throws Exception {
            return null;
        }
    }
    
    FutureTask futureTask = new FutureTask(new CallableTest());
````




<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
