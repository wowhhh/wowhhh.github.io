---
layout: post
title:  "Threadlocal的个人理解"
tags:   Java 多线程 面试 并发
date:   2020-12-16 20:00:10 +0800
categories: [Java]
---

#### ThreadLocal是什么

多线程访问同一个共享变量的时候容易出现并发问题，特别是多个线程对一个变量进行写入的时候，为了保证线程安全，一般使用者在访问共享变量的时候需要进行**额外的同步措施才能保证线程安全性**。
ThreadLocal是除了加锁这种同步方式之外的一种保证一种**规避多线程访问出现线程不安全的方法**.

当我们在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量这样就不会存在线程不安全问题。

也就是说ThreadLocal能够保证每个线程所访问的变量是自身线程才拥有的。

ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

```java
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 */
```

看下面这个例子：摘自->[用了三年 ThreadLocal 今天才弄明白其中的道理](https://mp.weixin.qq.com/s/vtLKqtHjAknbIV8RBojtbQ)

```java
public class ThreadLocalTest {
    public static void main(String[] args) {
        Task task = new Task();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> System.out.println(Thread.currentThread().getName() + " : " + task.calc(10))).start();
        }
    }

    static class Task {
        private int value;

        public int calc(int i) {
            value += i;
            return value;
        }
    }
}
```

运行结果：

![image-20201216220442982](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20201216220442982.png)

因为三个线程共用一个Task所以导致数据是累加的，如果说这个数据是让线程自己存储自己的话，就需要ThreadLocal来搭把手。

#### ThreadLocal用法

```java
import java.io;

class ThreadLocalTest{
    static ThreadLocal<String> localVar = new ThreadLocal<>();
    static void print(String str)
    {
        System.out.println(str+":"+localVar.get());
        localVar.remove();
    }
    public static void main(String[] args) {
        Thread t1  = new Thread(new Runnable() {
            @Override
            public void run() {
                //设置线程1中本地变量的值
                localVar.set("localVar1");
                //调用打印方法
                print("thread1");
                //打印本地变量
                System.out.println("after remove : " + localVar.get());
            }
        });

        Thread t2  = new Thread(new Runnable() {
            @Override
            public void run() {
                //设置线程1中本地变量的值
                localVar.set("localVar2");
                //调用打印方法
                print("thread2");
                //打印本地变量
                System.out.println("after remove : " + localVar.get());
            }
        });
        t1.start();
        t2.start();
    }
}
```

运行结果：

![image-20201216214401651](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20201216214401651.png)

可以看出来thread2改变了localVar的值以后还是不影响thread1

#### 源码解析(JDK 1.8)

Thread类里面又与ThreadLocal相关的变量：

```java
public class Thread implements Runnable {
 ......
//与此线程有关的ThreadLocal值。由ThreadLocal类维护
ThreadLocal.ThreadLocalMap threadLocals = null;

//与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
 ......
}
```

上面两个变量都是ThreadLocal类型的变量，默认情况下都为空，只有当线程调用ThreadLocal类的set或get方法的时候，才会创建他们，实际上调用这两个方法的时候，我们调用的是ThreadLocalMap类对应的get()、set()方法。

ThreadLocal类的结构图如下：

![image-20201216214742590](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20201216214742590.png)

ThreadLocal的set()方法：

```java
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

再看ThreadLocal的getMap方式是怎么实现的
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
CreareMap呢？
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

通过上面这些内容，我们足以通过猜测得出结论：**最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是`ThreadLocalMap`的封装，传递了变量值。** `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

而且**每个线程持有一个ThreadLocalMap对象**。每一个新的线程Thread都会实例化一个ThreadLocalMap并赋值给成员变量threadLocals，使用时若已经存在threadLocals则直接使用已经存在的对象。

那么问题来了，ThreadLocalMap是怎么样的结构呢？

```java
看构造方法如下：存储以ThreadLocal为 key ，Object 对象为 value 的键值对。
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```

**如果说一个线程有两个ThreadLocal变量的话，是如何存储的呢？**

都是存储在ThreadLocalMap中，以对应的ThreadLocal为key，以存储的值为value

![ThreadLocal数据结构](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/threadlocal%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)

这里就可以明白，每个线程只有一个ThreadLocalMap，但是ThreadLocal是有多个的（多对多）。ThreadLocalMap的Key是ThreadLocal的弱引用。

#### 大家都在提的内存泄露问题

我就直接复制[JavaGuilde](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E8%BF%9B%E9%98%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93?id=_34-threadlocal-%e5%86%85%e5%ad%98%e6%b3%84%e9%9c%b2%e9%97%ae%e9%a2%98)的话了

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。

所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。

这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。

ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。

**使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法**

再看下```set(),get(),remove()```的源码：

```java
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }    
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }     
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```



#### 面试题

待总结

#### 参考文章

[用了三年 ThreadLocal 今天才弄明白其中的道理](https://mp.weixin.qq.com/s/vtLKqtHjAknbIV8RBojtbQ)

[JavaGuild-ThreadLocal](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E8%BF%9B%E9%98%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93?id=_3-threadlocal)

[Java中的ThreadLocal详解](https://www.cnblogs.com/fsmly/p/11020641.html)