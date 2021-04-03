---
layout: post
title:  "Select、Poll、Epoll"
tags:   Java 
date:   2021-04-03 10:00:00 +0800
categories: [并发]


---

- ### select、poll、epoll
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
  epoll三个主要的操作：epoll_create、epoll_ctl、epoll_wait
  >epoll_create创建一个epoll fd和事件表, 在LINUX2.6.8以后是使用红黑树来管理epoll事件表，所以size没有太大作用
  >epoll_ctl操作上面创建的epoll事件表, 可以加入socket读写事件
>epoll_wait类似于以前的select和poll,得到发生的事件, 如socket可读可写