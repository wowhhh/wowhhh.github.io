---
layout: post
title:  "HashMap 源码分析"
tags:   Java 面试 源码分析
date:   2020-12-01 12:00:10 +0800
categories: [Java]

---

**基于 1.8**

https://segmentfault.com/a/1190000012926722

HashMap在JDK1.8之前是由数组加链表组成的，其中数组是主体，链表则是为了解决哈希冲突存在的。

1.8之后HashMap的组成多了红黑树，当链表的长度超过8的时候，就会把链表转换成红黑树。加快了索引速度。

底层的数据结构是数组，称之为哈希桶，每个桶里面放的是链表，链表中的每个节点就是哈希表中那个的每个元素。

因为哈希桶的数据结构是**数组**，所以就会涉及到扩容的问题。

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
          n |= n >>> 1;//现将n无符号右移1位，并将结果与右移前的n做按位或操作（相同为0，不同为1），结果赋给n
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
  //存放元素的数组，哈希桶
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
        final int hash; //记录Hash值，如果向HashMap中插入值，会通过Key计算hash值，判断该放到哪里合适
        final K key; //键
        V value; //值
        Node<K,V> next; //链表后置节点，指向下一个节点
		//构造方法
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
		//get()
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        //set
        public final String toString() { return key + "=" + value; }
		//重写HashCode()方法，可以得到每一个节点的hash值
        public final int hashCode() {
            //异或开运算，转换成二进制，相同为0，不同为1
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
		//设置新的value，返回旧的value
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
		//重写equals方法，判断当前Node与传入的Node是否相等
        public final boolean equals(Object o) {
            if (o == this)//通过地址判断
                return true;
            if (o instanceof Map.Entry) {//如果o是map里面的元素
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;//转换Map指定类型成元素
                if (Objects.equals(key, e.getKey()) &&//判断key和value是否都相等
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

Node类我们可以知道，这是一个单链表，每一个节点的Hash值，是将key的hashcode和value的hashcode异或得到的

再看TreeNode是如何实现的，这两个节点都有重要作用：

```java
    /**
    红黑树结构
    **/
	static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 父
        TreeNode<K,V> left;//左
        TreeNode<K,V> right;//右
        TreeNode<K,V> prev;    // 前序节点
        boolean red; //判断节点颜色，是否为红色
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * 返回根节点，因为根节点的pre是null
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

#### resize



#### 01：Get

这里我们主要关注的方法有：

```java
    public V put(K key, V value) {...}
	public void putAll(Map<? extends K, ? extends V> m) {...}
	public V get(Object key) {...}
	public V getOrDefault(Object key, V defaultValue) {...}
	public V remove(Object key) {...}
```

我们先看增加元素的方法：

先看get相关的方法

- get方法

  get方法主要的查找步骤是先定位键值对所在的桶的位置，然后对链表或者红黑树进行查找.

  那么是如何确定所在桶中的位置呢？实现的代码为：```first = tab[(n-1)&hash]```

  因为根据传入的hash值，其在桶中的下标就为index = (n-1)&hash，(n-1)&hash的计算方式就相当于hash%(n-1)，对n-1的长度进行取余运算，因为位运算的效率高，所以就采取了位运算的方式。

  还有一个hash()函数，目的是计算hash值，之前Node类里面已经重写了hashCode方法，但是这里并没有采用，而是重新计算了一次，因为有两个好处：

  1：防止高位数据不发挥作用，通过hash()方法，可以使得原来哈希值的高位和地位进行异或运算(相同为0，不同为1)，加大了地位信息的随机性，让高位数据也参与其中

  2：增加Hash的复杂度，原来的方式冲突率比较高

  ```java
      /**
      根据指定的key，获取其对应的value
      **/
  	public V get(Object key) {
          Node<K,V> e;
          return (e = getNode(hash(key), key)) == null ? null : e.value;
      }
  	//根据Hash值和对应的Key值获取节点
      final Node<K,V> getNode(int hash, Object key) {
          Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
          //1: 定位键值对所在桶的位置
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (first = tab[(n - 1) & hash]) != null) {
              if (first.hash == hash && // always check first node
                  ((k = first.key) == key || (key != null && key.equals(k))))
                  return first;
              if ((e = first.next) != null) {
                  // 2. 如果 first 是 TreeNode 类型，则调用黑红树查找方法
                  if (first instanceof TreeNode)
                      return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                   // 2. 对链表进行查找
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
  没啥将的，就是查找不到的话，按照指定的值返回
  ```

#### 02:put

寻找插入位置->桶是否为空->不为空就看是插入链表还是红黑树，需不需要扩容，需不需要链表转换成红黑树

- put方法

  ```java
      /**
      根据指定的key存放value，如果map之前已经包括了 此key，则替换value
      **/
  	public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
  ```

  再看putval

  1. 当桶数组 table 为空时，通过扩容的方式初始化 table
  2. 查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
  3. 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
  4. 判断键值对数量是否大于阈值，大于的话则进行扩容操作

  ```java
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          //tab存放 当前的哈希桶， p用作临时链表节点  
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          //如果当前哈希表是空的，代表是初始化
          if ((tab = table) == null || (n = tab.length) == 0)
              //那么直接去扩容哈希表，并且将扩容后的哈希桶长度赋值给n
              n = (tab = resize()).length;
          //如果当前index的节点是空的，表示没有发生哈希碰撞。 直接构建一个新节点Node，挂载在index处即可。
          //index 是利用 哈希值 & 哈希桶的长度-1，替代模运算，相当于将Hash值对长度-1取余
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {//否则 发生了哈希冲突。
              //e
              Node<K,V> e; K k;
              //如果哈希值相等，key也相等，则是覆盖value操作
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;//将当前节点引用赋值给e
              else if (p instanceof TreeNode)//红黑树暂且不谈
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {//不是覆盖操作，则插入一个普通链表节点
                  //遍历链表
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {//遍历到尾部，追加新节点到尾部
                          p.next = newNode(hash, key, value, null);
                          //如果追加节点后，链表数量》=8，则转化为红黑树
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      //如果找到了要覆盖的节点
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              //如果e不是null，说明有需要覆盖的节点，
              if (e != null) { // existing mapping for key
                  //则覆盖节点值，并返回原oldValue
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  //这是一个空实现的函数，用作LinkedHashMap重写使用。
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          //如果执行到了这里，说明插入了一个新的节点，所以会修改modCount，以及返回null。
  
          //修改modCount
          ++modCount;
          //更新size，并判断是否需要扩容。
          if (++size > threshold)
              resize();
          //这是一个空实现的函数，用作LinkedHashMap重写使用。
          afterNodeInsertion(evict);
          return null;
      }
  ```

#### 03：扩容

在 HashMap 中，桶数组的长度均是2的幂，阈值大小为桶数组长度与负载因子的乘积。当 HashMap 中的键值对数量超过阈值时，进行扩容。

HashMap 的扩容机制与其他变长集合的套路不太一样

HashMap 按当前桶数组长度的2倍进行扩容，阈值也变为原来的2倍（如果计算过程中，阈值溢出归零，则按阈值公式重新计算）。扩容之后，要重新计算键值对的位置，并把它们移动到合适的位置上去。

- resize

  ```java
      final Node<K,V>[] resize() {
          Node<K,V>[] oldTab = table;
          int oldCap = (oldTab == null) ? 0 : oldTab.length;
          int oldThr = threshold;
          int newCap, newThr = 0;
          //老桶的length>0，表示初始化过了
          if (oldCap > 0) {
              //如果已经是最大容量了，就将阈值也设置成最大值
              if (oldCap >= MAXIMUM_CAPACITY) {
                  threshold = Integer.MAX_VALUE;
                  return oldTab;
              }
              //新的容量是老容量*2，如果还小于最大容量
              //并且老容量大于默认容量 16，就将阈值也设置成原来的2倍
              else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                       oldCap >= DEFAULT_INITIAL_CAPACITY)
                  newThr = oldThr << 1; // double threshold
          }
          //老桶没有初始化，但是阈值不是0，这是因为通过构造函数调用的，构造函数当时传入initialCapacity，并且根据initialCapacity计算出了离他最近的2的幂次，赋值给了threshold
          else if (oldThr > 0) // initial capacity was placed in threshold
              newCap = oldThr;
          else { //调用无参构造的时候会进入这里，容量16，阈值为16*0.75
              newCap = DEFAULT_INITIAL_CAPACITY;
              newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
          }
          //新的阈值为0，这是因为扩容并且现在容量已经超过最大了，就需要对阈值进行重新计算，也就是上面第一个分支里面的第二个分支进来的
          //也有可能是老的容量为0，但是之前的阈值>0，也就是上面第二个分支进来的
          //第一个条件分支未计算 newThr 或嵌套分支在计算过程中导致 newThr 溢出归零
          if (newThr == 0) {
              float ft = (float)newCap * loadFactor;
              newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                        (int)ft : Integer.MAX_VALUE);
          }
          threshold = newThr;
          // 创建新的桶数组，桶数组的初始化也是在这里完成的
          @SuppressWarnings({"rawtypes","unchecked"})
              Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
          table = newTab;
          if (oldTab != null) {
              // 如果旧的桶数组不为空，则遍历桶数组，并将键值对映射到新的桶数组中
              for (int j = 0; j < oldCap; ++j) {
                  Node<K,V> e;
                  if ((e = oldTab[j]) != null) {
                      oldTab[j] = null;
                      if (e.next == null)
                          newTab[e.hash & (newCap - 1)] = e;
                      else if (e instanceof TreeNode)
                          // 重新映射时，需要对红黑树进行拆分
                          ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                      else { // preserve order
                          Node<K,V> loHead = null, loTail = null;
                          Node<K,V> hiHead = null, hiTail = null;
                          Node<K,V> next;
                           // 遍历链表，并将链表节点按原顺序进行分组
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
                           // 遍历链表，并将链表节点按原顺序进行分组
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

#### 04：Remove

第一步是定位桶位置，第二步遍历链表并找到键值相等的节点，第三步删除节点。

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
              // 1. 定位桶位置
              (p = tab[index = (n - 1) & hash]) != null) {
              Node<K,V> node = null, e; K k; V v;
              // 如果键的值与链表第一个节点相等，则将 node 指向该节点
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  node = p;
              else if ((e = p.next) != null) {
                  // 如果是 TreeNode 类型，调用红黑树的查找逻辑定位待删除节点
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
               // 3. 删除节点，并修复链表或红黑树
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
  



#### 04：modCount



#### 05 ：其他方法

https://blog.csdn.net/tuke_tuke/article/details/51588156

https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90?id=resize-%e6%96%b9%e6%b3%95