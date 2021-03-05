---
layout: post
title:  "HashMap 源码分析"
tags:   Java 面试 源码分析
date:   2020-12-01 12:00:10 +0800
categories: [Java]

---

**基于 1.8**

HashMap在JDK1.8之前是由数组加链表组成的，其中数组是主体，链表则是为了解决哈希冲突存在的。

1.8之后HashMap的组成多了红黑树，当链表的长度超过8的时候，就会把链表转换成红黑树。加快了索引速度。

#### 00 ：构造函数

还从构造函数先看起，HashMap总共提供给我们了四个构造函数，分别为：

```java
public HashMap(int initialCapacity, float loadFactor) {...}
public HashMap(int initialCapacity) {...}
public HashMap() {...}
public HashMap(Map<? extends K, ? extends V> m) {
```

先看无参构造

- 无参构造

  ```java
       /**
       * 当构造函数没有指定加载因子的大小时候，使用0.75
       */
      static final float DEFAULT_LOAD_FACTOR = 0.75f;
  	/**
       * HashTable的加载因子
       * @serial
       */
      final float loadFactor;
  	/**
       * 使用默认的容量大小、初始的加载因子，构造一个空的HashMap。
       	初始容量为16，初始的加载因子为0.75
       */
      public HashMap() {
          this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
      }
  ```

  无参构造就设置了下HashMap的加载因子为 0.75，其他啥也没干，加载因子后面会用来扩容使用，这里先不提。

  而且从这里我们还看不出来，保存HashMap数据的使用的是哪个数据结构。

- 含参构造 - int

  ```java
  /**
  	根据指定的容量以及默认的0.75的加载因子构造一个空的HashMap
  */
  public HashMap(int initialCapacity) {
      this(initialCapacity, DEFAULT_LOAD_FACTOR);
  }
  ```

  小老板不谈了，这里直接调用了另一个构造方法。

- 含参构造 - int , float

  ```java
      /**
       * 最大容量
      */
      static final int MAXIMUM_CAPACITY = 1 << 30;
      /**
       阈值 当实际大小(容量*填充因子)超过阈值时，会进行扩容
      **/
      int threshold;
  	/**
       * 根据指定的容量大小和加载因子，构造一个空的HashMap
       */
      public HashMap(int initialCapacity, float loadFactor) {
          if (initialCapacity < 0) //初始容量不能为0
              throw new IllegalArgumentException("Illegal initial capacity: " +
                                                 initialCapacity);
          if (initialCapacity > MAXIMUM_CAPACITY) //超过最大值，设为最大值
              initialCapacity = MAXIMUM_CAPACITY;
          if (loadFactor <= 0 || Float.isNaN(loadFactor)) //判断加载因子是否合法，isNan是用来判断是不是非数
              throw new IllegalArgumentException("Illegal load factor: " +
                                                 loadFactor);
          this.loadFactor = loadFactor; //将加载因子设置成指定的值
          this.threshold = tableSizeFor(initialCapacity);//根据设置的容量大小计算出阈值，阈值计算方式如下函数
      }
  	
  	
      /**
       >>>表示无符号右移，也叫逻辑右移，即若该数为正，则高位补0,比如7的二进制是111，7>>>2表示右移2位，变成001，即为1
       查所传的参数是否为2的幂次方，且不能为负数（负数变为1），且不能超过常量MAXIMUM_CAPACITY（超过变为MAXIMUM_CAPACITY），如果不为2的幂次方，将其变为，比cap大的最小的2的幂次方的值；
       */
      static final int tableSizeFor(int cap) {
          int n = cap - 1;
          n |= n >>> 1;//现将n无符号右移1位，并将结果与右移前的n做按位或操作，结果赋给n
          n |= n >>> 2;
          n |= n >>> 4;
          n |= n >>> 8;
          n |= n >>> 16;
          //中间过程的目的就是使n的二进制数的低位全部变为1，比如10，11变为11; 100，101，110，111变为111；
          return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
      }
  ```

  这个构造函数都干了什么呢？传入数据正确性的判断我们就不讨论了，后面首先重新设置加载因子，然后根据用户传入的容量大小，设置阈值的大小，阈值设置为与容量大小距离最近且为2的幂次方的数。

- 含参构造 - Map

  ```java
      public HashMap(Map<? extends K, ? extends V> m) {
          this.loadFactor = DEFAULT_LOAD_FACTOR;//设置加载因子为 0.75
          putMapEntries(m, false);//看下面的解析
      }
  ```

  ```java
  //存放元素的数组
  transient Node<K,V>[] table;
  
  //实现Map.putAll以及构造函数的功能,m为传入的要转换成HashMap的map，evict设置为false表示是构造时候使用，反之不是    
  final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
          int s = m.size();//m的大小
          if (s > 0) {
              if (table == null) { // table是不是初始化过
                  //将s除以负载因子+1可以得到HashMap所需的最大阈值
                  float ft = ((float)s / loadFactor) + 1.0F;
                  //如果计算得到的最大阈值大于最大值,则将t赋值为最大值
                  int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                           (int)ft : MAXIMUM_CAPACITY);
                  //如果t大于当前的临界值，重新根据t计算临界值
                  if (t > threshold)
                      threshold = tableSizeFor(t);
              }
              //已经初始化过，并且s内的个数已经大于了阈值，进行扩容
              else if (s > threshold)
                  resize();
              //将m中的所有元素都添加到Hashmap中
              for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                  K key = e.getKey();
                  V value = e.getValue();
                  putVal(hash(key), key, value, false, evict);
             }
          }
      }
  ```

构造函数我们看完了，主要就是容量、加载因子、临界值的设定。那到底是谁存储的的元素，谁设置的默认容量呢，如下是其他相关的变量定义。

```java
	/**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
	//存放元素的数组
    transient Node<K,V>[] table;
	//存放具体元素
	transient Set<Map.Entry<K,V>> entrySet;
```

构造函数中还出现了重要的方法**resize**，我们到扩容的时候再解析。

#### 00-01 插叙一手

在看读取增删相关的方法之前，我们先看看Node类是如何实现的：

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

再看TreeNode是如何实现的，这两个节点都有重要作用：

```java
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
        ....还有很多，就不看了，够用就行。。。
```



#### 01 ：读取增删

这里我们主要关注的方法有：

```java
    public V get(Object key) {...}
	public V getOrDefault(Object key, V defaultValue) {...}
	public V put(K key, V value) {...}
	public void putAll(Map<? extends K, ? extends V> m) {...}
	public V remove(Object key) {...}
```

先看get相关的方法

- get方法

  ```java
      /**
      根据指定的key，获取其对应的value
      **/
  	public V get(Object key) {
          Node<K,V> e;
          return (e = getNode(hash(key), key)) == null ? null : e.value;
      }
      final Node<K,V> getNode(int hash, Object key) {
          Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (first = tab[(n - 1) & hash]) != null) {
              if (first.hash == hash && // always check first node
                  ((k = first.key) == key || (key != null && key.equals(k))))
                  return first;
              if ((e = first.next) != null) {
                  if (first instanceof TreeNode)
                      return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                  do {
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          return e;
                  } while ((e = e.next) != null);
              }
          }
          return null;
      }
  ```

- getOrDefault

  ```java
      @Override
      public V getOrDefault(Object key, V defaultValue) {
          Node<K,V> e;
          return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
      }
  ```

- put

  ```java
      public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
  	
  	    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {
              Node<K,V> e; K k;
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              else if (p instanceof TreeNode)
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          p.next = newNode(hash, key, value, null);
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              if (e != null) { // existing mapping for key
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          ++modCount;
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
      }
  ```

- remove

  ```java
      public V remove(Object key) {
          Node<K,V> e;
          return (e = removeNode(hash(key), key, null, false, true)) == null ?
              null : e.value;
      }
  	
  	    final Node<K,V> removeNode(int hash, Object key, Object value,
                                 boolean matchValue, boolean movable) {
          Node<K,V>[] tab; Node<K,V> p; int n, index;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (p = tab[index = (n - 1) & hash]) != null) {
              Node<K,V> node = null, e; K k; V v;
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  node = p;
              else if ((e = p.next) != null) {
                  if (p instanceof TreeNode)
                      node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                  else {
                      do {
                          if (e.hash == hash &&
                              ((k = e.key) == key ||
                               (key != null && key.equals(k)))) {
                              node = e;
                              break;
                          }
                          p = e;
                      } while ((e = e.next) != null);
                  }
              }
              if (node != null && (!matchValue || (v = node.value) == value ||
                                   (value != null && value.equals(v)))) {
                  if (node instanceof TreeNode)
                      ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                  else if (node == p)
                      tab[index] = node.next;
                  else
                      p.next = node.next;
                  ++modCount;
                  --size;
                  afterNodeRemoval(node);
                  return node;
              }
          }
          return null;
      }
  ```

  

#### 02 ：扩容

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



#### 04：modCount



#### 05 ：其他方法

https://blog.csdn.net/tuke_tuke/article/details/51588156

https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90?id=resize-%e6%96%b9%e6%b3%95