---
layout: post
title:  "JVM总结个人向"
tags:   Java JVM
date:   2021-03-15 10:00:00 +0800
categories: [JVM]


---

### 内存区域

- JVM的主要组成部分和作用

  JVM整体来看分为：运行时数据区，类加载子系统、执行引擎以及本地方法接口（与native libraries交互，是其它编程语言交互的接口。）

  ![image-20210314090906995](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20210314090906995.png)

- JVM运行时数据区

  运行时数据区在JDK 1.8前后是有区别的，但都是分为线程私有与线程共有的区域。

  JDK1.8之前：共有的区域为：堆和方法区，私有的区域为：程序计数器，虚拟机栈和本地方法栈

  JDK1.8之后：永久代变为元空间，在jdk7及以前，习惯上把方法区称为永久代。jdk8开始，使用元空间取代了永久代

  ![image-20210314103634799](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210314103634799.png)

  所以线程间共享的是堆以及堆外内存（方法区、元空间或者永久代、代码缓存）

  - **程序计数器**

    程序计数器用于存储指向下一条指令的地址，也就是即将要指向的代码的地址；

    字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，比如：循环分支异常处理；

    多线程情况下，程序计数器用于记录当前线程执行的位置，从而当线程切换回来的适合知道允许到哪里了；

    程序计数器不会出现OutOfMemory，生命周期随着线程的创建而创建。

  - **虚拟机栈**

    虚拟机栈描述的是java方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧用于存储**局部变量表，操作数栈**，动态链接地址，方法返回地址等信息。每一个方法从开始执行到运行结束都对应一个入栈和出栈的过程；

    生命周期和线程一致；

    不存在垃圾回收问题；

    栈中可能存在的异常：StackoverflowError：线程请求分配的栈容量大于了jvm虚拟机栈运行的最大容量；OutOfMemoryError：创建新的线程是，没有足够的内存区创建对应的虚拟机栈；

  - 本地方法栈

    本地方法栈和虚拟机栈的作用非常相似，只不过本地方法栈描述的是Native方法执行的内存模型。

- 方法中定义的局部变量是否线程安全

  不一定，主要还是看其他线程能不能访问到

  因为如果A线程提供的方法的返回值是某个变量的地址，是引用类型，也会被B线程的某个方法访问到，出现线程不安全。

- JVM如何执行方法的调用的

  先讨论重载和重写方法：方法重载在编译阶段就能确定下来，方法重写是在运行时候才能确定。java编译器是根据传入参数的类型来选取重载的方法的，jvm识别方法是依赖于方法描述符，是由方法的参数类型以及返回类型所构成的。

  再简述五种调用指令：invokestatic调用静态方法，invokespecial调用私有实例方法，invokevirtual调用非私有实例方法，invokeinterface调用接口方法，invokedynamic调用动态方法

  解释invokestatic和invokespecial：invokestatic指令和invokespecial指令调用的方法称为非虚方法，静态方法，私有方法，final方法，实例构造器，父类方法都是非虚方法，此类方法在编译器就能确定调用的具体版本了，并且运行时候不可变

  解释invokeviture和invokeinterface：在绝大多数情况下，JVM 需要在执行过程中，根据调用者的动态类型来确定具体的目标方法。这两者的调用称为虚方法调用，JVM 采用了一种空间换时间的策略来实现动态绑定。它为每个类生成一张方法表，用于快速定位目标方法，这个发生在类加载的准备阶段。

  方法表本质上是一个数组，它有两个特性，首先是子类方法表中包含父类方法表中所有的方法，其次是子类方法在方法表中的索引，与它所重写的父类方法的索引值相同。我们知道，方法调用指令中的符号引用会在执行之前解析为实际引用。对于静态绑定的方法调用而言，实际引用将指向具体的方法，对于动态绑定而言，实际引用则是方法表的索引值。

- JDK1.7和1.8，堆的内存结构的异同

  JDK7以及以前:年轻代，老年代，永久区

  JDK8以后：年轻代，老年代，元空间

  年轻代可以分为Eden空间，Survivor0空间以及Survivor1空间

- JVM中堆栈的区别

  栈是运行时的单位，而堆是存储的单位，即栈解决程序的运行问题，即程序如何运行，如何处理数据。栈解决的是数据存储的问题，即数据怎么存放，放在哪里。

- 说一下内存模型

  JMM 内存模型是用来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各个平台下都能达到一致的内存访问效果。

  Java 内存模型规定了所有的共享变量都是存储在主内存，每个线程还有自己的工作内存，线程的工作内存保存了该线程使用到的共享变量的主内存副本拷贝，线程对变量的操作都必须在工作内存中进行，而不能直接读写主内存中的变量，不同的线程之间也无法直接访问对方工作内存中的数据，线程间变量值的传递均需要主内存来完成。

- 内存分配回收的策略

  1：对象优先在Eden区分配:如果Edan区的内存不足，会触发一次Minor GC

  2：大对象直接进入老年代：大对象指的是需要连续大量内存空间的Java对象

  3：长期存活的对象进入老年代：虚拟机采用了分代的思想来管理内存，内存回收就必须识别哪些对象在新生代哪些在老年代。虚拟机设置了一个对象年龄计数器如果。对象在 Eden 出生并经过一次 Minor GC 后仍然存活，并且能被 Survivor 容纳的话，将会被移到 Survivor 空间中，并且对象年龄设置为 1.对象每在 Survivor 区熬过一次 Minor GC，年龄就会增加 1。当年龄增加到一定程度，默认是 15，就将会晋升到老年代中。

- Minor GC/Major GC/Full GC

  Minor GC是针对年轻代进行的，触发的条件是Eden区满了，Survivor区不会主动触发，是被动的GC。

  老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC。

  Full GC/Major GC 指发生在老年代的 GC，出现 Full GC 经常会伴随着至少一次的 Minor GC，Full GC 一般会比 Minor GC 慢十倍以上。

- HotSpot方法区的演变

  ![image-20210314104119912](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210314104119912.png)

  ![image-20210314104130131](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20210314104130131.png)

- 为什么使用元空间代替永久代

  1：为永久代设置空间大小是很难确定的。 在某些场景下，如果动态加载类过多，容易产生Perm区的O0M。

  2：元空间和永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制

- 为什么要有两个survivor区

  确保内存是连续的，**新生代gc比较频繁、对象存活率低，用复制算法在回收时的效率会更高，也不会产生内存碎片**。但复制算法的代价就是要将内存折半，为了不浪费过多的内存，就划分了两块相同大小的内存区域survivor from和survivor to。在每次gc后就会把存活对象给复制到另一个survivor上，然后清空Eden和刚使用过的survivor。

- Stringtable为什么调整

  因为永久代的回收效率低，只有full GC才会触发，导致了StringTable回收效率不高。确保了能及时进行回收。

- 深拷贝和浅拷贝

- 对象创建的过程

  1：new

  遇到一条new指令的时候，首先检查元空间的常量池中是否有此类的符号引用

  2：加载

  检查上述的符号引用是否已经被加载、解析和初始化，如果没有，就执行类加载流程

  3：分配内存

  类加载完毕知乎，就会在堆中为对象分配内存，内存大小在加载的时候已经确定了。分配方式主要有两种：

  ​	1：指针碰撞：内存工整的情况下使用指针碰撞，意思是所有用过的内存在一边，空闲的内存在另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针向空闲那边挪动一段与对象大小相等的距离罢了。

  ​	2：空闲列表：内存不规整，虚拟机需要维护一个列表，使用空闲列表分配，虚拟机维护了一个列表，记录上哪些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。

  ​	在分配内存的时候还需要考虑到并发的问题，虚拟机采用了两种方式解决：1：CAS，保证指针更新操作的原子性，2：使用线程私有的分配缓冲区

  4：初始化分配到的空间

  内存分配结束之后，虚拟机将分配到的内存空间都初始化为零值。保证了全局变量不需要初始化也能用。

  5：设置对象的对象头

  对行头包括Markword和类型指针

  MarkWorld有：哈希值，GC分代年龄，锁状态的标志，线程持有的锁，偏向线程ID，偏向时间戳

  类型指针主要是虚拟机通过这个指针判断是哪个类的实例

  6：执行init方法进行初始化

  Java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。

  

- 对象的内存布局

  对象的内存布局包括：对象头，实例数据与对其填充

  对象头主要是Mark World和类型指针，MarkWorld有哈希值，GC分代年龄，锁状态，线程持有的锁，偏向线程ID和偏向时间戳

  类型指针是表明是哪个对象类型

  示例数据是对象真正存储的有效信息，包括程序代码中定义的各种类型的字段（包括从父类继承下来的和本身拥有的字段） 规则

  ![image-20210314113004935](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210314113004935.png)

- 对象的访问定位

  Java 程序需要通过栈上的 reference 数据来操作堆上的具体对象，由于 reference 类型在 Java 虚拟机规范中只规定了一个指向对象的引用，并没有规定这个引用应该通过什么方式去定位和访问堆中的对象，所以对象访问方式也是取决于虚拟机实现而定。

  对象的访问有句柄访问和直接指针两种方式。

  句柄访问：java栈中的reference指向堆中的句柄池的地址，句柄池里面维护了对象实例和对象类型数据的地址，优点是对象地址变化的时候，reference不需要变化。

  直接指针：java栈中的reference指向堆中实例的地址，实例里面记录了对象类型数据的地址。

- StringTable

  https://juejin.cn/post/6844904192209846280



## GC

- 如何判断对象是否死亡

  引用计数法：给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的。缺点是难以处理对象之间的循环引用问题

  可达性分析法：通过GC Roots为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。

  常见的GC Roots：

  虚拟机栈中引用的对象

  - ➢比如：各个线程被调用的方法中使用到的参数、局部变量等。

  本地方法栈内JNI（通常说的本地方法）引用的对象

  方法区中类静态属性引用的对象

  - ➢比如：Java类的引用类型静态变量

  方法区中常量引用的对象

  - ➢比如：字符串常量池（string Table） 里的引用

  所有被同步锁synchroni zed持有的对象

  Java虚拟机内部的引用。

  - ➢基本数据类型对应的Class对象，一些常驻的异常对象（如： NullPointerException、OutOfMemoryError） ，系统类加载器。

- 介绍一下强引用、软引用、弱引用、虚引用

  强引用：默认的引用类型，只要强引用的对象是可触及的，垃圾收集器永远不会回收掉被引用的对象。

  软引用：内存不足即回收

  弱引用：发现即回收

  虚引用：对象回收跟踪

- 如何判断一个常量是废弃常量

  注意：

  >1. **JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代**
  >2. **JDK1.7 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是 hotspot 中的永久代** 。
  >3. **JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)**

  假如在字符串常量池中存在字符串 "abc"，如果当前**没有任何 String 对象引用该字符串常量**的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

- 如何判断一个类是无用类

  不是必然被回收的，满足以下三个条件有可能被回收：

  1：该类的所有实例都被回收，也就是Java堆中已经不存在该类的任何实例

  2：该类的classloader被回收

  3：该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

- 垃圾收集算法

  当成功分出内存中存活的对象和死亡的对象之后，GC接下来的任务

  - 标记-清除

    标记清楚法的执行过程：标记清除法进行两项工作，第一个是标记，第二个是清除，标记工作即从根节点开始遍历，标记出所有被引用的对象，一般是在对象头中标记为可达对象；清除工作就是对堆内存进行线性的遍历，如果发现对象头没有标记为可达对象，就将其回收

    缺点：

    效率低

    进行GC的时候，需要停止整个应用程序

    这种方式清理出来的空闲内存是不连续的，产生内存碎片，需要维护空闲链表。

    清除也不是真的置空，而是把需要清除的对象地址保存在空闲列表里面，下次有新对象需要加载的时候，判断垃圾的位置空间是否足够，足够的话就直接存放

  - 标记-复制

    标记-复制算法的执行过程：将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

    优点：内存连续

    缺点：两倍的内存空间；

  - 标记-整理

    执行过程：1：标记阶段从开始的根节点标记所有被引用的对象；2：将所有存活对象压缩到内存的一端，按照顺序存放 3：清理边界外的空间

    标记-整理算法的最终效果等同于标记-清除算法最后对内存做了碎片化的整理

  - 分代收集

    分代收集的产生是因为不同对象的生命周期是不一样的，因此不同生命周期的对象可以采取不同的收集方法，以提高回收效率。

    **比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。**

- HotSpot为什么分为新生代和老年代

  同上分代收集

- 常见垃圾回收器

  Serial(old)、ParNew(old)、Paraller Scavenge、CMS、G1、ZGC

  - 新生代回收器：Serial、ParNew、Parallel Scavenge
  - 老年代回收器：Serial Old、Parallel Old、CMS
  - 整堆回收器：G1

- CMS

  执行过程也是先初始标记，并发标记，最终标记以及并发清除。

  使用的清除算法为标记-清除算法

- G1

  **特点：**

  并行与并发：G1 收集器仍然可以通过并发的方式让 java 程序继续执行。G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作。

  分代收集：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。

  空间整合: 与 CMS 的“标记-清理”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。

  可预测的停顿：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

  **执行流程：**

  初始标记：仅仅标记以下GC Roots能直接关联到的对象，这个阶段需要停顿线程，但是耗时很短。

  并发标记：从GC Roots开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象，这阶段耗时较长，但是可与用户程序并发执行。

  最终标记：对用户线程做另一个短暂的暂停，用于处理在并发标记阶段新产生的对象引用链变化。

  筛选回收：负责更新 Region 的统计数据，对各个 Region 的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划。



## 类加载

- 类加载机制与类加载过程

  虚拟机把class类加载到内存，并且对其验证，准备以及解析最后初始化，形成可以被虚拟机使用的java类型。

  从class文件到加载到内存中的类，到类卸载出内存为止，总共经历了七个阶段：

  加载->验证->准备->解析->初始化->使用->卸载；其中验证、准备和解析三个阶段统称为链接

  **加载阶段：**查找并加载类的二进制数据，生成Class的实例，1：通过一个类的全限定名来获取定义此类的二进制字节流，2：将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构（就是将类的二进制数据流转换成方法区内的数据结构）。

  加载阶段是开发人员可控性最强的阶段，因为开发人员可以自定义类加载器。

  对于数组而言，情况有所不同，数组类本身不通过类加载器创建，它是由 Java 虚拟机直接创建，但数组的元素类型仍然需要依靠类加载器去创建。

  **验证阶段：**链接阶段的第一步，这一阶段的目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，加载的字节码是合法、合理并符合规范的，并且不会危害虚拟机自身的安全。它包括文件格式校验、元数据校验、字节码校验等。

  **准备阶段：**正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。需要注意的是，这时候进行内存分配的**仅仅包含类变量，不包括实例变量**，实例变量将会在对象实例化时随着对象一起分配在 Java 堆上。其次，这里所说的**变量初始值是该数据类型的零值**。

  **解析阶段：**虚拟机将常量池内的符号引用替换为直接引用的过程，得到类、字段、方法在内存中的指针或者偏移量。**通过解析操作，符号引用就可以转变为目标方法在类中方法表中的位置，从而使得方法被成功调用**

  **初始化阶段：**初始化阶段是执行类构造器 <clinit>() 方法的过程。<clinit>() 方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的。虚拟机会保证一个类的 <clinit>() 方法在多线程环境中被正确的加锁同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 <clinit>() 方法，其他线程都需要阻塞等待，这也是静态内部类能实现单例的主要原因之一。



- 什么是类加载器，类加载器有哪些

  实现通过类的完全限定名获取该类的二进制字节流的代码块叫做类加载器

  1：启动类加载器(Bootstrap Classloader)用来加载java核心类库，无法被java程序直接引用

  2：扩展类加载器(extensions class loader):它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。主要负责加载目录 `%JRE_HOME%/lib/ext`

  3：系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 包和类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它

  4：用户自定义类加载器，通过继承 java.lang.ClassLoader类的方式实现

- 什么是双亲委派机制

  **工作过程：**如果一个类加载器收到类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一层次的类加载器都是如此，因此所有的类加载请求最终都应该传送给顶层的启动类加载器中，只有当父类加载器反馈自己无法完成这个加载请求时，子加载器才会尝试自己去加载。

  **好处：**避免了类的重复加载，相同的类文件被不同的类加载器产生的是不同的类的。

  使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是 Java 类随着它的类加载器一起具备了一种带有优先级的层次关系。比如 Object 类，无论哪个类加载器去加载，应用程序各种加载器环境中都是同一个类，同时也避免了重复加载。而且，双亲委派模型也保证了 Java 程序的稳定运作。

  **双亲委派机制的实现：**双亲委派模型的实现相对简单，代码都集中在 ClassLoader 的 loadClass 方法中先检查是否已经被加载过了，如果没加载则先调用父加载器的 loadClass 方法，若父加载器为空则使用默认的启动类加载器作为父加载器。如果父加载器加载失败，抛出 ClassNotFoundException 异常，然后调用自己的 findClass 方法进行加载。

  ```java
                  try {
                      if (parent != null) {
                          c = parent.loadClass(name, false);//调用父类
                      } else {
                          c = findBootstrapClassOrNull(name);//父类为空，使用启动类加载器
                      }
                  } catch (ClassNotFoundException e) { //父类加载失败
                      // ClassNotFoundException thrown if class not found
                      // from the non-null parent class loader
                  }
  
                  if (c == null) { //调用自己的findclass
                      // If still not found, then invoke findClass in order
                      // to find the class.
                      long t1 = System.nanoTime();
                      c = findClass(name);
                      ...
                  }
  ```

  