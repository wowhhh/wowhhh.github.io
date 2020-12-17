---
layout: post
title:  "线程池的个人理解"
tags:   Java 多线程 面试 并发
date:   2020-12-17 20:00:10 +0800
categories: [Java]
---

## 线程池是什么

使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。

* **线程池：**其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

#### 线程池的好处

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

## 线程池的使用

Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

### Executors创建线程池

- `public static ExecutorService newFixedThreadPool(int nThreads)`

  返回线程池对象。返回的是固定大小的线程池(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

> ```
> @param nThreads the number of threads in the pool
> @return the newly created thread pool
> ```

```java
return new ThreadPoolExecutor(nThreads, nThreads,
                              0L, Time Unit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>());
```

- `public static ExecutorService newSingleThreadExecutor()`

  创建只有一个线程的线程池

  ```java
  return new FinalizableDelegatedExecutorService
      (new ThreadPoolExecutor(1, 1,
                              0L, TimeUnit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>()));
  ```

> @return the newly created single-threaded Executor 

- `public static ExecutorService newCachedThreadPool()`

  创建一个不设置线程数上限的线程池，任何提交的任务都将立即执行。

```java
return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                              60L, TimeUnit.SECONDS,
                              new SynchronousQueue<Runnable>());
```

从上面三个简单的创建方式可以看到，都是调用了`ThreadPoolExecutor`的构造方法，构造方法的参数如下：

```java
public ThreadPoolExecutor(int corePoolSize, // 线程池长期维持的线程数，即使线程处于Idle状态，也不会回收。
                          int maximumPoolSize, // 线程数的上限
                          long keepAliveTime,// 超过corePoolSize的线程的idle时长，超过这个时间，多余的线程就会被回收
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue, // 任务的排队队列
                          ThreadFactory threadFactory, // 新线程的产生方式、
                         RejectedExecutionHandler handler) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         threadFactory, handler);
}
```



获取到了一个线程池ExecutorService 对象，那么怎么使用呢，在这里定义了一个使用线程池对象的方法如下：

* `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行

  > Future接口：用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

使用线程池中线程对象的步骤：

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

Runnable实现类代码：

~~~java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
~~~

线程池测试类：

~~~java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}
~~~

#### 构造方法创建线程池

摘自[JavaGuide](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93?id=%e5%9b%9b-%e9%87%8d%e8%a6%81threadpoolexecutor-%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)

```MyRunnable.java```

```java
import java.util.Date;

public class MyRunnable implements Runnable
{
    private String message;
    public MyRunnable(String message)
    {
        this.message = message;
    }
    
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" Start Time = "+ new Date());//输出开始此线程的时间
         processCommand();//暂停
        System.out.println(Thread.currentThread().getName()+" End Time = "+ new Date());//输出结束此线程的时间
    }
    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    @Override
    public String toString() {
        // TODO Auto-generated method stub
        return this.message;
    }
}
```

```ThreadPoolTest.java```

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolTest {
    private static final int CORE_POOL_SIZE = 5;//指定核心线程数
    private static final int MAX_POOL_SIZE = 10;//指定最大线程数
    private static final int QUEUE_CAPACITY = 100;//用于指定ArrayBlockingQueue任务队列的容量
    private static final Long KEEP_ALIVE_TIME = 1l;//等待时间为1s，s是由TimeUnit.SECONDS指定的


    public static void main(String[] args) {
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(CORE_POOL_SIZE, MAX_POOL_SIZE, KEEP_ALIVE_TIME, TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(QUEUE_CAPACITY),
        new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 10; i++) {
            Runnable worker = new MyRunnable(""+i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

输出结果如下：

```
pool-1-thread-2 Start Time = Thu Dec 17 15:18:53 CST 2020
pool-1-thread-3 Start Time = Thu Dec 17 15:18:53 CST 2020
pool-1-thread-1 Start Time = Thu Dec 17 15:18:53 CST 2020
pool-1-thread-4 Start Time = Thu Dec 17 15:18:53 CST 2020
pool-1-thread-5 Start Time = Thu Dec 17 15:18:53 CST 2020
//手动分割
pool-1-thread-2 End Time = Thu Dec 17 15:18:58 CST 2020
pool-1-thread-3 End Time = Thu Dec 17 15:18:58 CST 2020  
pool-1-thread-5 End Time = Thu Dec 17 15:18:58 CST 2020  
pool-1-thread-1 End Time = Thu Dec 17 15:18:58 CST 2020  
pool-1-thread-4 End Time = Thu Dec 17 15:18:58 CST 2020
//手动分割
pool-1-thread-1 Start Time = Thu Dec 17 15:18:58 CST 2020
pool-1-thread-5 Start Time = Thu Dec 17 15:18:58 CST 2020
pool-1-thread-3 Start Time = Thu Dec 17 15:18:58 CST 2020
pool-1-thread-2 Start Time = Thu Dec 17 15:18:58 CST 2020
pool-1-thread-4 Start Time = Thu Dec 17 15:18:58 CST 2020
//手动分割
pool-1-thread-4 End Time = Thu Dec 17 15:19:03 CST 2020
pool-1-thread-1 End Time = Thu Dec 17 15:19:03 CST 2020
pool-1-thread-5 End Time = Thu Dec 17 15:19:03 CST 2020
pool-1-thread-2 End Time = Thu Dec 17 15:19:03 CST 2020
pool-1-thread-3 End Time = Thu Dec 17 15:19:03 CST 2020
Finished all threads
```

**从代码的输出能看出来什么？**

>我们在代码中模拟了 10 个任务，我们配置的核心线程数为 5 、等待队列容量为 100 ，所以每次只可能存在 5 个任务同时执行，剩下的 5 个任务会被放到等待队列中去。当前的5个任务中如果有任务被执行完了，线程池就会去拿新的任务执行。

## 源码解析(JDK 1.8)

### 解析ThreadPoolExecutor

主要分析的是类```ThreadPoolExecutor```的源码

```java
结构如下：
    public class ThreadPoolExecutor extends AbstractExecutorService {...}
	public abstract class AbstractExecutorService implements ExecutorService {...}
	public interface ExecutorService extends Executor {...}
	public interface Executor {...}
```

##### ThreadPoolExecutor的构造方法

```java
public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               ) {
    ...
}
```

建议看这里更详细：[ThreadPoolExecutor类分析](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93?id=_31-threadpoolexecutor-%e7%b1%bb%e5%88%86%e6%9e%90)

### 解析Executor

#### 面试题

待总结

#### 参考文章

[JavaGuild-线程池](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E8%BF%9B%E9%98%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93?id=_4-%e7%ba%bf%e7%a8%8b%e6%b1%a0)

[线程池最佳实践](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/%E6%8B%BF%E6%9D%A5%E5%8D%B3%E7%94%A8%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5?id=%e7%ba%bf%e7%a8%8b%e6%b1%a0%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5)

[线程池的总结](https://snailclimb.gitee.io/javaguide/#/./docs/java/multi-thread/%E6%8B%BF%E6%9D%A5%E5%8D%B3%E7%94%A8%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5?id=%e7%ba%bf%e7%a8%8b%e6%b1%a0%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5)