---
layout: post
title:  "Select、Poll、Epoll"
tags:   Java 
date:   2021-04-03 10:00:00 +0800
categories: [并发]


---

### select、poll、epoll
- select
1：有最大连接数的限制，因为fd的数量是写死了，fd_setsize = 32
2：fd_set还需要从内核空间拷贝到进程空间里面，也是比较耗时的
3：轮询去判断每个fd是不是准备好，也耗时
>最大并发限制: 1024个描述符
>内核/用户空间的fd_set数据拷贝
>遍历整个fd_set集合效率低，集合越大浪费的时间越多 

- poll
poll主要为了解决select的两个问题，所以在结构上面做了改进：
第一个问题：文件描述符数量(fd_setsize = 32)太小, 而且数值是使用宏写死的,这样在32位机器上最大文件描述符数量只有32*32=1024
第二个问题：文件描述符集(fd_set)这种值-结果参数的api设计不是很好, select系统调用的时候要分别传读set,写set，更多事件不好细分
改进：poll系统调用使用了pollfd数据结构来表示事件数组，没有了fd_setsize的限制,同时支持更多的事件类型
```c
struct pollfd {
  int fd; // 需要检查的fd
  short events; // 该fd感兴趣的事件
  short revents; // 该fd当前发生的事件
}
```
poll仍然存在的问题：
1：拷贝问题，poll这里是用数组来存储的，还是会面临从内核到用户空间的拷贝
2：还是需要遍历整个数组
>fd数组(不管是fd_set还是pollfd)都要在用户空间和内核空间之间来回拷贝
>被监控的fd数组有事件的时候，需要遍历整个数组
- epoll
https://www.cnblogs.com/lojunren/p/3856290.html
epoll三个主要的操作：epoll_create、epoll_ctl、epoll_wait
>epoll_create，添加epoll描述符。创建一个epoll fd和事件表, 在LINUX2.6.8以后是使用红黑树来管理epoll事件表，所以size没有太大作用
>epoll_ctl 加入事件，操作上面创建的epoll事件表, 可以加入socket读写事件
>epoll_wait 类似于以前的select和poll,得到发生的事件, 如socket可读可写

与select相比，epoll分清了频繁调用和不频繁调用的操作。例如，epoll_ctrl是不太频繁调用的，而epoll_wait是非常频繁调用的。这时，epoll_wait却几乎没有入参，这比select的效率高出一大截，而且，它也不会随着并发连接的增加使得入参越发多起来，导致内核执行效率下降。

epoll的三大关键要素：mmap、红黑树、链表。
mmap： epoll是通过内核与用户空间mmap同一块内存实现的。mmap将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址（不管是用户空间还是内核空间都是虚拟地址，最终要通过地址映射映射到物理地址），使得这块物理内存对内核和对用户均可见，减少用户态和内核态之间的数据交换。内核可以直接看到epoll监听的句柄，效率高。

红黑树： 红黑树将存储epoll所监听的套接字。上面mmap出来的内存如何保存epoll所监听的套接字，必然也得有一套数据结构，epoll在实现上采用红黑树去存储所有套接字，当添加或者删除一个套接字时（epoll_ctl），都在红黑树上去处理，红黑树本身插入和删除性能比较好，时间复杂度O(logN)。

链表：双向链表，通过epoll_ctl函数添加进来的事件都会被放在红黑树的某个节点内，所以，重复添加是没有用的。当把事件添加进来的时候时候会完成关键的一步，那就是该事件都会与相应的设备（网卡）驱动程序建立回调关系，当相应的事件发生后，就会调用这个回调函数，该回调函数在内核中被称为：ep_poll_callback,这个回调函数其实就所把这个事件添加到rdllist这个双向链表中。一旦有事件发生，epoll就会将该事件添加到双向链表中。那么当我们调用epoll_wait时，epoll_wait只需要检查rdlist双向链表中是否有存在注册的事件，效率非常可观。这里也需要将发生了的事件复制到用户态内存中即可。