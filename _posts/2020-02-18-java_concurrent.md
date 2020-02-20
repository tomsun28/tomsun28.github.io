---
layout: post
title:  Java Concurrent Learn
date: 2020-02-18
tag: java
---

<br>
<br>

## Java Concurrent Learn   



- - -

<br>

对java并发的一些学习,之前总结在幕布上,为了加深印象,这里重新学习下。  

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

### 同步控制  

* ``` synchronized Object.wait() Object.notify ```  
synchronized同步关键字，对类，方法，对象进行修饰，修饰后获取对应监视器才能进入修饰的临界区  
Object.wait()，Object.notify()，Object.notifyAll()方法必须在获取到监视器对象后才能调用，即要写在synchronized内部  
Object.wait()之后会将当前线程放入对象的wait set中，释放获取的监视器对象  
Object.notify()会从wait set随机对一线程唤醒，Object.notifyAll()会从wait set所有线程唤醒，不会立即释放获取的监视器，待结束临界区才释放，唤醒不代表被唤醒线程继续执行,其还需要争抢获取监视器才能继续执行    

````
/**
 * @author tomsun28
 * @date 19:49 2020-02-19
 */
public class Demo {

    private static final Logger LOGGER = LoggerFactory.getLogger(Demo.class);
    private final static Demo demo = new Demo();

    public static void waitDemo() {
        synchronized (demo) {
            try {
                // 调用wait方法必须获取对应的监视器,在wait之后,释放监视器
                LOGGER.info("获取demo对象监视器1");
                demo.wait();
                LOGGER.info("释放demo对象监视器1");
                Thread.sleep(2000);
                LOGGER.info("释放demo对象监视器2");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void notifyDemo() {
        // 从等待队列中唤醒一个
        synchronized (demo) {
            LOGGER.info("获取demo对象监视器2");
            demo.notify();
            LOGGER.info("释放demo对象监视器3");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LOGGER.info("释放demo对象监视器4");
        }
    }

    public static void main(String[] args) {
        new Thread(() -> waitDemo()).start();
        new Thread(() -> notifyDemo()).start();
    }
}

````

* ``` ReentrantLock Condition ```  
重入锁(可重复获取的锁)，重入锁内部的Condition  
ReentrantLock lock几次,就需要unlock几次，lock之后需要紧跟tay finally代码，finally第一行就要unlock  
Condition功能用法和Object.wait notify差不多，只不过其搭配重入锁使用，不用在使用前获取对象监视器  
Condition.await()会释放当前拥有的锁，并挂起当前线程  
Conditon.singal()会随机唤醒当前await的线程，Condition.singalAll()会唤醒所有，当然同Object.notify()唤醒不代表被唤醒线程可以继续执行，其还需要争抢获取到锁后继续执行  

````
/**
 * @author tomsun28
 * @date 22:15 2019-10-27
 */
public class ReenterLockCondition {

    public static void main(String[] args) throws InterruptedException{
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();
        new Thread(() -> {
            // 先获取重入锁
            System.out.println("0 - 获取锁");
            reentrantLock.lock();
            try {
                System.out.println("0 - 释放锁,挂起线程");
                // 同 Object.wait相似 其和ReentrantLock绑定,这里就是释放锁,挂起线程
                condition.await();
                System.out.println("0 - 等待锁释放,thread is fin");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }).start();
        System.out.println("1 - 睡4秒");
        Thread.sleep(4000);
        System.out.println("1 - 等待获取锁");
        reentrantLock.lock();
        try {
            System.out.println("1 - condition 唤醒线程");
            condition.signal();
            Thread.sleep(4000);
            System.out.println("1 - 睡4秒释放锁");
        } finally {
            reentrantLock.unlock();
        }
    }
}
````

* ``` Semaphore ```  
信号量-不同于锁,其可以控制进入临界区的线程数量,锁只能是一个线程进入临界区  
构造函数可以初始化信号数量，semaphore.acquire()消耗一个信号，semaphore.release()释放一个信号，当所剩的信号为0时则不允许进入临界区  

````
/**
 * @author tomsun28
 * @date 22:53 2019-10-27
 */
public class SemaphoreDemo {

    public static void main(String[] args) {
        final Semaphore semaphore = new Semaphore(5);
        final AtomicInteger atomicInteger = new AtomicInteger(1);
        Runnable thread1 = () -> {
            try {
                semaphore.acquire();
                Thread.sleep(2000);
                System.out.println(atomicInteger.getAndAdd(1));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
        };
        ExecutorService executorService = Executors.newFixedThreadPool(20);
        IntStream.range(0,20).forEach(i -> executorService.execute(thread1));
        executorService.shutdown();
    }
}
````


* ``` ReetrantReadWriteLock ```  
读写锁，之前的重入锁还是synchronized，对于读写都是同样的加锁  
ReadWriteLock把读和写的加锁分离，读读不互斥，读写互斥，写写互斥提高了锁性能，
需要注意，当有读锁存在时，写锁不能被获取，在大量一直在进行读锁的时候，会造成写锁的饥饿，即写锁一直获取不到锁  

````
/**
 * @author tomsun28
 * @date 21:35 2020-02-19
 */
public class ReadWriteLockDemo {

    public static void main(String[] args) {
        ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
        new Thread(() -> {
            readWriteLock.readLock().lock();
            readWriteLock.readLock().lock();
            try {
                System.out.println("获取2读锁");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
              e.printStackTrace();
            } finally {
                readWriteLock.readLock().unlock();
                readWriteLock.readLock().unlock();
                System.out.println("释放2读锁");
            }
        }).start();

        new Thread(() -> {
            readWriteLock.readLock().lock();
            try {
                System.out.println("获取3读锁");
                Thread.sleep(6000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readWriteLock.readLock().unlock();
                System.out.println("释放3读锁");
            }
        }).start();

        new Thread(() -> {
            readWriteLock.writeLock().lock();
            try {
                System.out.println("获取1写锁");
            } finally {
                readWriteLock.writeLock().unlock();
                System.out.println("释放1写锁");
            }
        }).start();
    }

}


````


* ``` StampedLock ```  
上面的读写锁有一个缺点，即可能写饥饿，StampedLock解决这个问题  
使用乐观读 - 使用标记戳来判断这次读的过程是否有写的过程，若有则重新读取或转化为悲观读  

````
/**
 * @author tomsun28
 * @date 21:57 2020-02-19
 */
public class StampedLockDemo {

    public static void main(String[] args) {
        StampedLock stampedLock = new StampedLock();
        AtomicInteger flag = new AtomicInteger(0);
        new Thread(() -> {
            long writeStamped = stampedLock.writeLock();
            try {
                System.out.println("获取写锁,休息1.006秒");
                Thread.sleep(1006);
                System.out.println("当前值为: " + flag.getAndAdd(1));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally{
                stampedLock.unlockWrite(writeStamped);
                System.out.println("释放写锁");
            }

        }).start();

        new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long stamp = stampedLock.tryOptimisticRead();
            int readValue = flag.intValue();
            while (!stampedLock.validate(stamp)) {
                System.out.println("重试乐观读");
                stamp = stampedLock.tryOptimisticRead();
                readValue = flag.intValue();
            }
            System.out.println("乐观读为:" + readValue);
        }).start();
    }
}

````

* ``` CountDownLatch ```  
并发减数拦截器，await()拦截阻塞当前线程，当减数器为0时通行  

````
/**
 * @author tomsun28
 * @date 23:35 2019-10-27
 */
public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(10);
        AtomicInteger atomicInteger = new AtomicInteger(1);
        Thread threadd = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println(" thred check complete" + atomicInteger.getAndIncrement());
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        IntStream.range(0, 10).forEach(i -> executorService.submit(threadd));
        countDownLatch.await();
        System.out.println("fin");
    }
}

````


* ``` CyclicBarrier ```  
栅栏，功能类似于ConutDownLatch，不过可以循环重置处理  
ConutDownLatch是减数器为0时触发，并重置减数器循环等待下一次触发，并且可以提供触发方法  

````
/**
 * @author tomsun28
 * @date 23:59 2019-10-27
 */
public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(10, () -> {
            System.out.println("this cyclic ok");
        });
        AtomicInteger atomicInteger = new AtomicInteger(1);
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        IntStream.range(0, 100).forEach(i -> {
            executorService.execute(() -> {
                try {
                    Thread.sleep(1000);
                    System.out.println(atomicInteger.getAndIncrement());
                    cyclicBarrier.await();
                } catch (BrokenBarrierException | InterruptedException e) {
                    e.printStackTrace();
                }
            });
        });
        executorService.shutdown();
    }
}


````

* ``` LockSupport ```  
同步工具类，比起wait notify要方便很多  
LockSupport.park()是在当前线程阻塞挂起  
LockSupport.unpark(thread)唤醒指定线程  

````
/**
 * @author tomsun28
 * @date 00:39 2019-10-28
 */
public class LockSupportDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
           try {
               LockSupport.park();
               LockSupport.park();
               System.out.println("fin");
           } catch (Exception e) {
               e.printStackTrace();
           }
        });
        thread.start();
        Thread.sleep(3000);
        System.out.println("begin");
        LockSupport.unpark(thread);
        Thread.sleep(3000);
        LockSupport.unpark(thread);

    }
}

````


### 线程池  

* Executors 提供的线程池  
  1. ``` newFixedThreadPool(threadNum) ``` 最大与核心线程数量为threadNum的linked队列线程池   
  2. ``` newSingleThreadExecutor() ```最大与核心线程数量为1的linked队列线程池  
  3. ``` newCachedThreadPool ``` 核心数量为0，最大线程数量为Integer.MAX_VALUE的线程池，线程数量会跟插入的任务同步增长   
  4. ``` newSingleThreadScheduledExecutor() ``` 核心数量为1，最大线程数量为Integer.MAX_VALUE的延迟队列线程池  
  5. ``` newWorkStealingPool() ``` ForkJoin线程池   

* ``` ThreadPoolExecutor ```  自定义构造线程池  
  1. ``` corePoolSize ``` 核心线程数量   
  2. ``` maxPoolSize ``` 最大线程数量  
  3. ``` keepAliveTime ``` 超出核心线程数的线程的空闲存活时间  
  4. ``` unit ```  时间单位   
  5. ``` blockingQueue ``` 存放未被执行的任务线程的阻塞队列，当队列满时，才会去从核心数量开始往最大线程数方向新增运行线程   
  6. ``` threadFactory ```  线程创建工厂   
  7. ``` rejectedHandler ``` 任务满拒绝策略   

* ``` ForkJoinPool ```  
  > ``` ForkJoinTask ```   
  >> ``` RecursiveTask<T> ```  
  >> ``` RecursiveAction ```  


### 并发容器  

* 非并发容器转并发处理 ``` Collections.synchronziedXXX() ```
其本质是在容器外层加了```synchronized(mutex)```  

* 并发```map - CpncurrentHashMap - 1.7 segment 1.8 synchronzied cas```  
* 并发```ArrayList - RCU - read copy update - CopyOnWriteArrayList```  
* 并发```Queue - ConcurrentLinkedQueue``` 非阻塞队列  
* 阻塞队列 ``` BlockingQueue - ReentrantLock Condition ```
  1. ```ArrayBlockingQueue```数组阻塞队列
  2. ```LinkedBlockingQueue```列表阻塞队列
  3. ```SynchronousQueue```此阻塞队列无容量立即转发
  4. ```PriorityBlockingQueue```优先级阻塞队列
  5. ```DelayedWorkQueue```延迟阻塞队列  

### 原子类   

* ```AtomicBoolean```  
* ```AtomicInteger```  
* ```AtomicLong```  
* ```LongAddr```  
* ```AtomicRefrence```  
* ```AtomicStampedRefreence```  

### other  

* ```ThreadLocal```线程本地变量  
* ```volatile```可见关键字  
* ```Future```非阻塞模式  
* ```CompletableFuture```非阻塞  

<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
